# Corvus-Loopis / Corvus-Wake
## Technical Architecture Report

**Version:** 2.0 (Integrated)
**Classification:** Internal design document
**Systems covered:** Corvus-Loopis (internal reasoning-state rollback substrate) + Corvus-Wake (external side-effect compensation ledger)

---

## 1. Executive Summary

Corvus-Loopis is a fault-tolerant execution substrate for autonomous, multi-step LLM agents. When a reasoning trajectory hits deadlock, unrecoverable tool failure, or a safety violation, it rewinds execution to the nearest verified checkpoint instead of terminating the job or blindly retrying in-context.

Corvus-Wake is its paired sibling system, addressing what Corvus-Loopis explicitly cannot: actions that have already touched the outside world (sent an email, charged a card, written to an external database) before a rollback was triggered. No internal state rollback can undo a witnessed external effect — Corvus-Wake exists to compensate, contain, and honestly flag that boundary rather than pretend it doesn't exist.

**Core design discipline carried through every component:** separate what is cheap and reversible from what is expensive or irreversible, and never let one silently masquerade as the other. This principle recurs at every layer of the system and is the single most load-bearing idea in the whole architecture.

---

## 2. Scope and Honest Boundaries

### 2.1 What this system is
A rollback substrate for the **reversible, pre-commitment** portion of agentic execution — internal reasoning, local/sandboxed state, and tool calls with a real inverse.

### 2.2 What this system is explicitly not
- Not a system that can undo real-world effects with no inverse (sent messages, executed trades, physical actuation, published content).
- Not a hardware-level VM/container snapshot system (Firecracker/gVisor-style). Those tools revert everything *inside* a sandbox boundary — filesystem, process memory — but do not and cannot reach outside it. A reverted VM does not un-charge a card or un-send an email, because the effect never lived inside the VM; only the code that requested it did. This system does not claim to solve a different problem than hardware snapshotting solves — it targets the same wall from the other side, with an explicit compensation ledger instead of a silent gap.

### 2.3 Relationship to prior art
The core rewind concept is not claimed as novel in isolation — **GA-Rollback** (Generator-Assistant Stepwise Rollback, EMNLP 2025) already demonstrates stepwise agent rollback with assistant-triggered correction. Google's conversation-graph checkpointing patent (US12242811B2) covers checkpoint/resume for dialog agents. The genuinely novel contribution claimed by this architecture is narrower and more specific: the **fusion of paged/radix-indexed KV-cache virtualization with incremental graph repair (LPA*/D*-Lite) and a formal external-effect compensation ledger**, applied together to govern autonomous multi-step agents. Each individual technique below has prior art cited where known; the composition is the claim.

---

## 3. System Architecture Overview

```
                    ┌─────────────────────────────────────┐
                    │      PRE-ROLLBACK PREVENTION         │
                    │  Constraint Gate  │  Skip-Gate       │
                    └───────────────┬───────────────────────┘
                                    │
                    ┌───────────────▼───────────────────────┐
                    │      TRIGGER CLASSIFICATION            │
                    │   🟢 Green │ 🟡 Yellow │ 🔴 Red         │
                    │   (Cumulus decaying-risk ledger feeds  │
                    │    escalation across zones)            │
                    └───────────────┬───────────────────────┘
                                    │ (Red / unrecoverable Yellow)
                    ┌───────────────▼───────────────────────┐
                    │         REWIND MECHANISM               │
                    │  Prune → Causality Log → LPA*/D*-Lite  │
                    │  repair → Rollback to Lagna Checkpoint │
                    │  → Confidence-scaled negative          │
                    │    constraint injection                │
                    └───────────────┬───────────────────────┘
                                    │
                    ┌───────────────▼───────────────────────┐
                    │   REPEATED-FAILURE FALLBACK            │
                    │        (Arbiter Ladder)                │
                    └───────────────┬───────────────────────┘
                                    │
        ┌───────────────────────────┴───────────────────────────┐
        │                                                        │
┌───────▼────────┐                                    ┌──────────▼─────────┐
│ STATE SUBSTRATE │                                    │   CORVUS-WAKE       │
│  DAG bookkeeping │◄──────── checkpoint refs ────────►│  Compensation Ledger│
│  + Paged/Radix   │                                    │  Tool tiering (A/B/C)│
│  KV-cache (P_Radix)│                                  │  LIFO saga engine   │
└───────┬────────┘                                    │  Irrevocable        │
        │                                              │  Waypoints          │
┌───────▼────────┐                                    └──────────┬─────────┘
│ CALIBRATION LOOP │                                              │
│ (firewalled,      │                                    (shares Arbiter ladder
│  thresholds only) │                                     for compensation retry)
└──────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  GOVERNANCE LAYER (outside the DAG's own logic, always active)   │
│  Sentinel Gate (deterministic, fail-closed safety policy)         │
│  Two-mode split: Strict vs. Fast — never mixed at runtime          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Corvus-Loopis — Component Breakdown

### 4.1 Pre-Rollback Prevention Layer

**Purpose:** reduce how often a rewind is ever needed, by stopping problems before they become failures.

| Component | Mechanism | Origin / Real-world analog |
|---|---|---|
| **Constraint Gate** | Hard rules (schema, banned actions, policy) compiled into an automaton (DFA/PDA) that masks disallowed tokens/actions *before* they execute. On SaaS-backed models without logit access, degrades gracefully to provider-native structured-output/JSON-schema enforcement first, then to post-hoc validation only as last resort. | Grammar-constrained decoding (Outlines, SGLang-style compilers); CAPELLA's Constraint Gate design |
| **Skip-Gate** | Cheap pre-check ("is this branch actually unrecoverable?") before paying for a full rewind. Asymmetric payoff: a miss costs only the check; a hit saves the entire rollback. | Verplaca's Envelope Strategy (2.1) — cheap sortedness check before invoking an expensive vendor kernel |

**Design note:** these do not replace rollback. They narrow how often the expensive path (§4.3) is invoked.

### 4.2 Trigger Classification — Three-Zone Model

Replaces the original binary "trigger → full rewind" design.

| Zone | Condition | Action |
|---|---|---|
| 🟢 **Green** | Executing normally | Continue |
| 🟡 **Yellow** | Locally recoverable drift or error | In-place correction attempt, no full rewind |
| 🔴 **Red** | Deadlock / unrecoverable tool failure / safety violation | Full rewind to nearest Lagna Checkpoint (§4.3) |

**Cumulus Ledger (risk accumulation):** every action posts a risk weight to a running, windowed, decaying sum keyed by session/actor/resource — not just its own isolated action. When the accumulated sum crosses threshold, the zone escalates (Yellow→Red) even without one single dramatic failure event. This catches slow, compounding semantic drift that a purely reactive, single-event trigger would miss entirely.

*Origin: Pandemonium v4's Cumulus Demon — cumulative-risk ledger pattern borrowed from financial "structuring" detection (catching many small transactions that sum to a reportable one).*

### 4.3 State Representation — The Dual-Channel Split

This is the central architectural fix of the whole system, and the direct answer to the original unresolved gap: **"context/KV-state rollback cost is unsolved."**

Two explicitly separate problems, formally distinguished so neither is silently assumed to behave like the other:

| Channel | Contents | Cost profile | Mechanism |
|---|---|---|---|
| **C_Vector — Durable declarative state** | DAG bookkeeping: branch identity, cost, causality log, checkpoint metadata | Kilobyte-scale. Cheap. Copy-on-write native. | Versioned delta pointers — standard CoW tree, same as Git-style diffing |
| **P_Radix — Volatile execution state** | The model's actual working memory: KV-cache tensors | Gigabyte-scale. Historically the unsolved, expensive part. | **Paged, radix-indexed KV-cache** (see §4.3.1) |

*Origin of the split itself: CAPELLA v1→v2's §2.10 correction, which was originally criticized for conflating cheap tree-repair with expensive model-internal-state repair, and fixed by explicitly naming them as two different problems.*

#### 4.3.1 The KV-Cache Fix: Paged, Radix-Indexed Checkpointing

**The insight:** stop treating a checkpoint as "a snapshot of the whole KV tensor to store." Instead:

- KV-cache is split into fixed-size **blocks** (pages), not one contiguous tensor.
- Blocks are stored in a **radix tree**, keyed by token sequence.
- Two branches sharing a prefix **share the same physical blocks**, reference-counted — no copy exists until the branches diverge.
- New tokens after a divergence point allocate new blocks only; everything before it stays shared and uncopied.

This is literal copy-on-write at the token/attention-state granularity — not a metaphor for it.

| Old framing | New framing |
|---|---|
| Checkpoint = stored KV-cache snapshot | Checkpoint = **pointer into the radix tree** at a token position — no data copied |
| Rewind = restore saved snapshot | Rewind = **reset cursor** to checkpoint node, decrement refcounts on abandoned blocks, let GC reclaim them |
| Rollback cost ≈ size of full context | Rollback cost ≈ **O(blocks created since the checkpoint)** |
| N speculative branches = N full copies | N branches = **N pointers sharing prefix blocks** until actual divergence |

**Origin / prior art:** this is the same mechanism underlying **vLLM's PagedAttention** and **SGLang's RadixAttention**, both built to solve a structurally identical problem (many concurrent requests sharing a prompt prefix). CAPELLA v1's own component glossary flagged radix trees as "legitimate elsewhere — token-level KV-cache reuse" while rejecting them for its own (different) use case; that parenthetical is the seed of this fix.

**Honest limit, stated explicitly, not hidden:** sharing is only free while blocks remain resident. Under memory pressure, eviction reclaims cold blocks — including ones a stale checkpoint still points to. At that point, rollback degrades gracefully from O(pointer reset) back to **recompute-from-nearest-surviving-prefix** — the original expensive fallback. This is a real boundary condition, not a solved problem in all cases.

**Mitigations for the eviction boundary (tunable, not required):**
- Eviction priority favors keeping Lagna Checkpoint boundary blocks resident longer than ordinary mid-branch blocks.
- Optional KV compression/quantization for checkpoint blocks specifically (trades minor quality for longer residency under pressure).

**Disambiguation forced by this fix:** "context embeddings" in the original pitch was ambiguous between two different cost profiles. This split makes it explicit:
- **KV-cache** → solved (with the stated eviction boundary) by the mechanism above.
- **External retrieval/memory embeddings** (RAG-style) → a separate, much cheaper problem — versioned vector-store entries, naturally CoW-friendly, never needed this fix.

### 4.4 Rewind Mechanism

Sequence fired on a Red-zone trigger:

1. **Detect** — Red-zone trigger confirmed (hard failure, or Cumulus threshold crossed)
2. **Prune** — failed branch removed from the DAG
3. **Extract causality log** — structured record of *why* the branch failed
4. **LPA\*/D\*-Lite repair** — recompute only the affected branch's remaining value by reusing prior computation, instead of a full re-plan from scratch. Applied directly over the active cursors of the paged radix prefix tree (§4.3.1), giving a provable reduction in recomputation cost versus naive backtracking or full MCTS-style re-search.
5. **Rollback** — cursor reset to nearest Lagna Checkpoint (see §4.3.1 — this is the pointer-reset operation, not a data copy)
6. **Negative constraint injection** — a corrective signal is added to context to block re-entry into the failed trajectory. **Injection strength is scaled by confidence in the causality-log diagnosis** — a low-confidence diagnosis produces a softer constraint plus flagged monitoring, rather than a hard block based on an uncertain read of the failure cause.

*Origin: step 6's confidence-gating pattern is transferred from CAPELLA's Field Confidence Score (steering strength scales down when the underlying prediction is low-confidence) — a pattern transfer, not a literal component reuse, since CAPELLA's confidence source (embedding-field distance) doesn't exist in this context.*

**Status flag:** negative-constraint confidence-scaling is a transferable *pattern*, not yet a benchmarked mechanism — flagged as hypothesis pending measurement, same discipline Verplaca applied to its own unvalidated incremental patch cache.

### 4.5 Repeated-Failure Fallback — The Arbiter Ladder

Closes a gap the original design left completely open: what happens when the same checkpoint fails rewind repeatedly?

1. One additional attempt using a **materially different retry strategy** (not a repeat of the same failed approach)
2. Escalate to a **structurally distinct resolution path** (different tool or method entirely)
3. **Human confirmation**, if available within the time budget
4. **Default to the most conservative safe state**, with the outcome **explicitly flagged unresolved** — never silently presented as success

*Origin: Pandemonium v4's Arbiter Demon — Debate Protocol's deadlock-resolution ladder, itself modeled on distributed-systems tie-breaking and incident-response fail-safe defaults.*

**Shared use:** this same ladder is reused by Corvus-Wake for failed compensation attempts (§5.5) — one escalation mechanism, two callers, rather than a duplicated bespoke handler in each system.

### 4.6 Calibration Loop (Firewalled)

Recalibrates rewind-trigger thresholds (Yellow/Red boundary, Cumulus decay window, Arbiter-ladder retry cap) using real outcomes — did a Yellow-zone in-place correction actually hold, or fail downstream anyway?

**Hard firewall:** write access limited strictly to threshold values. **Zero access** to the Constraint Gate's hard rules or Sentinel Gate safety policy — those remain human-authored and non-learned by design, permanently.

*Origin: Pandemonium v4's Genesis Demon — outcome-driven recalibration of router confidence from production traffic, firewalled identically (weights only, never safety-critical thresholds). Mirrors CAPELLA's Field Calibration Loop.*

### 4.7 Governance Structure

| Mechanism | Function | Origin |
|---|---|---|
| **Sentinel Gate** | Deterministic, fail-closed safety-policy check sitting **outside** the DAG's own self-policing logic — the reasoning engine cannot approve its own Ring-2-equivalent actions | Pandemonium's Sentinel Gate |
| **Two-mode split (Strict / Fast)** | Strict = always full checkpoint/rewind (default, safety-critical). Fast = opt-in, skips checkpoints under high confidence. **No shared code path silently branches between them at runtime.** | VALKRI's Secure/Turbo compiled-separately-never-mixed principle |
| **Online miss-rate tracking** | Auto-disables a retry/correction strategy that is losing more than it saves | Verplaca's per-bucket miss-rate disable logic |
| **Decision-table lookup for trigger classification** | Deliberately not a trained model — "a model would cost more than the kernels it's choosing between" | Verplaca's selector-mechanism design principle |

### 4.8 Scaling (Deferred / Optional)

**HNSW-style approximate indexing** for checkpoint/branch lookup — only relevant once concurrent checkpoint count is large enough that linear lookup becomes a bottleneck. Not required for initial builds.

---

## 5. Corvus-Wake — Compensation Ledger

### 5.1 Why It Exists

Corvus-Loopis reverts internal reasoning state. It does **not**, and structurally cannot, revert effects already witnessed by the outside world. Hardware-level sandbox snapshotting (Firecracker/gVisor) does not solve this either — it reverts what's inside the sandbox boundary, not what already left it. Corvus-Wake is the explicit, honest answer to that boundary: compensate what can be compensated, and make what can't be compensated into a visible, hard fact instead of a silent gap.

### 5.2 Tool Classification (Fail-Closed by Default)

Every tool an agent can call is classified at integration time — never guessed at runtime.

| Tier | Definition | Rollback behavior |
|---|---|---|
| **A — Contained** | Effect never leaves the sandbox (local file, in-memory state, local DB) | Already covered by Corvus-Loopis's own state rollback (§4.3–4.4); no Wake Log entry needed |
| **B — Compensable** | Has a genuine inverse action (cancel booking, refund, delete created record) | Wake Log entry created; inverse fires automatically on rewind |
| **C — Witnessed** | No inverse exists (sent email, human notification, physical actuation, published content, executed trade) | Cannot be rewound — becomes a hard floor (§5.4) |

**Default for an unregistered tool: Tier C.** The system never assumes reversibility; a developer must actively supply a compensator function to earn Tier B status. This is the same fail-closed discipline as Sentinel Gate, applied to tool integration.

**Known friction from this default:** wrapping a pre-existing agent codebase with unregistered side-effecting tools will surface every one of them as an Irrevocable Waypoint immediately — there is no such thing as zero-config wrapping once a single unregistered side effect exists in the graph. **Mitigation:** an **audit/dry-run mode** — wrap the graph, replay against historical/logged traces without enforcing, and emit a report of every tool that would trip Tier C before the developer switches to enforcing mode. This converts a silent freeze into a one-time, visible configuration pass.

**Zero-config defaults for common tools:** the library ships with pre-registered Tier B/C classifications for widely used integrations (e.g., Stripe, SendGrid, Twilio, common SQL patterns) to reduce the classification tax — while still failing closed to Tier C for anything unrecognized.

### 5.3 Biasing Toward the Last Reversible Sub-Step

Where an external API supports it, execution is routed through a hold/authorize step before a capture/commit step (payment auth-then-capture, seat-hold-then-ticket, draft-then-publish) whenever available. The execution graph treats the irreversible commit as its own distinct node, and the rewind trigger is biased to fire before that node whenever the failure signal was already visible earlier. This is the cheapest lever in the entire system — most real damage comes from committing a step earlier than the failure was detectable, not from the absence of a compensation mechanism.

### 5.4 The Wake Log

An **append-only** ledger, structurally separate from the DAG. Every Tier B/C action writes: the checkpoint ID it executed under, the tool called, its compensator reference (Tier B only), the external confirmation ID, and a timestamp.

**This ledger is never itself rewound.** It is the one permanent record in the whole system, specifically because the DAG above it is allowed to be pruned and rewritten.

**Irrevocable Waypoint:** the first Tier C action downstream of a rewind target becomes a hard floor. Corvus-Loopis may still rewind reasoning *after* that point, but a rewind attempting to erase the waypoint itself must surface as an **explicit irrecoverable divergence** — internal state and external world are now permanently forked — rather than silently reporting success.

### 5.5 Compensation Execution

On a rewind whose target has Wake Log entries downstream:

- **Tier A** — nothing to do; already covered by state rollback.
- **Tier B** — compensators fire in **LIFO (reverse) order** — a later action may depend on an earlier one still existing (e.g., cannot refund a sub-booking before cancelling the reservation it's attached to). Standard saga-pattern ordering.
- **Tier C** — halts the rewind at the Irrevocable Waypoint (§5.4); does not proceed silently past it.

**Idempotency requirement:** compensators must be safe to call more than once (a rewind retry, or an Arbiter-ladder retry, may invoke the same compensation call twice). Enforced via idempotency keys on the compensation call itself — Stripe-style — not just the original action.

**Failed compensation:** routes through the same **Arbiter Ladder** defined in §4.5 (retry with different strategy → structurally distinct fallback → human → conservative default, explicitly flagged unresolved). One shared escalation mechanism across both systems.

---

## 6. Cross-Cutting Design Principles

These recur across every component above and are stated once here as the architecture's governing discipline, rather than repeated per-section.

| Principle | Where it shows up |
|---|---|
| **Never conflate cheap bookkeeping state with expensive execution state** | §4.3 dual-channel split (C_Vector vs P_Radix); root-caused from CAPELLA's own v1→v2 correction |
| **Fail-closed by default, always** | Sentinel Gate (§4.7); Tier C default for unregistered tools (§5.2) |
| **No silent mode-switching** | Strict/Fast split, compiled separately (§4.7) |
| **Prevention before correction, correction before rollback** | Prevention Layer (§4.1) → Yellow-zone in-place correction (§4.2) → Red-zone full rewind (§4.4), in strictly ascending cost order |
| **Flag unresolved rather than fake success** | Arbiter Ladder's final step (§4.5); Irrevocable Waypoint's explicit divergence report (§5.4) |
| **State speculative/unbenchmarked components honestly** | Confidence-scaled constraint injection (§4.4) explicitly flagged as pattern-transfer, not validated mechanism |
| **Wrap and reduce call frequency/cost rather than out-engineer vendor internals** | Skip-Gate (§4.1); Envelope Strategy origin from Verplaca |
| **One escalation mechanism, reused, not duplicated per subsystem** | Arbiter Ladder shared by Corvus-Loopis (§4.5) and Corvus-Wake (§5.5) |

---

## 7. Provenance Map

Every borrowed mechanism, traced to its source system, for audit purposes.

| Component | Source | Adaptation |
|---|---|---|
| Three-zone (Green/Yellow/Red) classification | CAPELLA v2 | Direct structural adoption |
| Constraint Gate | CAPELLA v2 | Direct adoption, extended with SaaS-degradation path |
| LPA\*/D\*-Lite branch repair | CAPELLA v1/v2 | Direct adoption, applied over radix-tree cursors specifically |
| Field Confidence Score → scaled correction strength | CAPELLA v2 | Pattern transfer only (confidence source differs) |
| DAG-repair ≠ KV-cache-repair separation | CAPELLA v1→v2 correction (§2.10) | Direct structural lesson, root of §4.3's entire design |
| Cumulus Demon (decaying risk ledger) | Pandemonium v4 | Direct adoption |
| Arbiter Demon (deadlock ladder) | Pandemonium v4 | Direct adoption, shared across both systems |
| Genesis Demon (firewalled recalibration) | Pandemonium v4 | Direct adoption |
| Sentinel Gate | Pandemonium v3 | Direct adoption |
| Two-mode compiled split (Strict/Fast) | VALKRI (Secure/Turbo) | Direct structural adoption |
| Skip-Gate, Envelope Strategy, miss-rate auto-disable | Verplaca | Direct adoption |
| Decision-table over trained model for selection | Verplaca | Direct adoption |
| Paged/radix KV-cache checkpointing | vLLM (PagedAttention), SGLang (RadixAttention) | Adopted as the literal mechanism for Lagna Checkpoints |
| Stepwise agent rollback (general concept) | GA-Rollback, EMNLP 2025 | Cited prior art, not the novel claim |
| Conversation-graph checkpointing (general concept) | US12242811B2 | Cited prior art, not the novel claim |
| LIFO saga compensation, idempotency keys | Classical distributed-systems saga pattern; Stripe API design | Direct adoption |

---

## 8. Known Open Gaps (Honestly Carried Forward)

1. **KV-cache eviction boundary** — rollback is O(pointer reset) only while blocks remain resident; under memory pressure it degrades to recompute. Mitigations (§4.3.1) reduce but do not eliminate this.
2. **Constraint Gate on SaaS-backed models** — without logit-level access, falls back to provider-native structured outputs, then post-hoc validation as last resort; loses the hard automaton-masking guarantee available on self-hosted inference.
3. **Confidence-scaled constraint injection** — unbenchmarked pattern transfer, not a validated mechanism.
4. **Tool classification burden** — despite zero-config defaults for common tools (§5.2), any unrecognized tool still requires explicit developer classification; fail-closed default means this cannot be silently skipped.
5. **Irreversible side effects remain irreversible** — Corvus-Wake compensates and contains; it does not and cannot undo a witnessed external effect. This is a physical boundary, not an engineering gap to be closed later.
6. **Deployment tiering** — the full paged-KV-cache win (§4.3.1) is only available to teams running self-hosted inference (vLLM/SGLang); SaaS-API-backed deployments run a degraded Lite tier without the memory-level guarantee. This is a real market-segmentation constraint, not solved by packaging alone.

---

## 9. Deployment / Packaging Model (Summary)

| Layer | Delivery | Availability |
|---|---|---|
| Corvus-Loopis-Lite (DAG bookkeeping, three-zone triggers, Constraint Gate w/ degradation path, Arbiter Ladder, Calibration Loop, Corvus-Wake) | `pip install corvus-loopis` | Any LLM API, no special infra required |
| Decorator/context-manager interface | `@checkpoint(...)` / `with rollback_zone():` | Included in base package |
| Framework adapters | `corvus_loopis.adapters.wrap_langgraph(...)` etc. | Included; ships with audit/dry-run mode to surface Tier C friction before enforcement |
| Corvus-Loopis-Core (paged/radix KV-cache checkpointing, §4.3.1) | `pip install corvus-loopis[vllm]`, optional, off by default | Requires self-hosted vLLM/SGLang backend |

---

*End of report.*
