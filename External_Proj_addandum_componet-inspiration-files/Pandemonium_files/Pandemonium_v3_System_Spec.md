# Pandemonium v3 — Full System Specification

**Status:** Design specification, unimplemented. Every mechanism below is architecturally defined; none has been coded, trained, or benchmarked. Section 11 lists what building it actually requires.
**Version lineage:** v1 (Sovereign Weave / Apex Loom, speculative) → v2 (grounded rewrite, 8 core fixes) → v2.1 (8 new components, audited) → **v3 (this document — 3 structural fixes + 2 targeted capability upgrades, re-audited)**
**Document owner:** design record, consolidated from architecture, audit, and gap-analysis sessions.

---

## 1. Executive Summary

Pandemonium v3 is a multi-stage reasoning-orchestration architecture for LLM-driven agentic systems. It routes subtasks to the cheapest method confident enough to handle them, verifies output before trusting it, coordinates multiple sub-agents through a shared workspace instead of blind parallelism, and — critically — moves authority over irreversible real-world actions **outside the reasoning engine entirely**, into a deterministic policy layer it cannot reason its way past.

It is not a novel algorithm in the research sense. It is a systems-integration design that borrows well-established graph-search, planning, and multi-agent primitives (detailed in §6) and applies them to LLM orchestration, with every design decision traceable to a specific failure mode it was built to close. A structured audit against 64 adversarial and real-world scenarios (§9) scored it at rough parity with documented current industry practice overall (7.03 vs. 6.98 / 10), ahead in the two categories it specifically targets (long-horizon agentic work, multi-agent coordination), and honestly behind in one category it deliberately traded away (hard real-time latency).

---

## 2. Design Principles

1. **Never worse than the baseline.** Every added stage is optional and reversible. The system must never underperform plain single-pass reasoning; it can only add value or degrade gracefully to that floor.
2. **Every mechanism has a stated cost.** No component is presented as a free win. Where a component buys accuracy or safety, the specific thing it spends (latency, token cost, complexity, new attack surface) is documented alongside it.
3. **Reasoning-based safeguards are not enforcement.** A model checking its own output twice is a quality mechanism, not a control mechanism. Anything irreversible is gated by policy that sits structurally outside the model (§7).
4. **Latency is a spendable resource, not a fixed constraint (v3 change).** Earlier versions optimized for speed by default. v3 assumes correctness is worth waiting for unless a specific deployment explicitly opts into a low-latency profile — which most of this architecture is not built for and should not pretend to be (see §9, Category F).
5. **Degradation must be explicit and ordered**, never silent. If a stage fails or exceeds budget, the system drops one rung down a defined fallback ladder rather than stalling or guessing.

---

## 3. System Architecture Overview

```
                              ┌─────────────────────┐
 problem ──────────────────▶  │   Budget Governor    │  (allocates & reclaims per-stage budget)
                              └──────────┬───────────┘
                                         ▼
                              ┌─────────────────────┐
                              │  Router + Triage      │  (task-shape classifier, input screening)
                              └──────────┬───────────┘
                          confidence < floor │ confidence ≥ floor
                                ▼            ▼
                         Plain CoT   ┌───────────────────────────┐
                         (safe        │ Constraint-Checked         │
                          default)    │ Decomposition               │  (feasibility pre-check on subtask DAG)
                                     └──────────┬─────────────────┘
                                                ▼
                                   ┌─────────────────────────────┐
                                   │      Blackboard Commons        │◀──────────┐
                                   │  (shared workspace, per-task)  │           │
                                   └──────────┬──────────────────┘            │
                                              ▼                                │
                     ┌────────────────────────────────────┐                  │
                     │  Bounded Adaptive Recursion            │                  │
                     │  (single agent) OR Debate Protocol       │──────────────────┘
                     │  (multi-agent, contested subtasks)       │   (posts result + confidence)
                     └──────────┬───────────────────────────┘
                                ▼
                downstream failure? ──yes──▶ Retrospective Rewiring (can backtrack the DAG, not just patch forward)
                                │no
                                ▼
                     ┌─────────────────────────────┐
                     │ Verification (Vantage Ledger)   │  (2 or 3 disjoint chains, tiered by stakes)
                     └──────────┬───────────────────┘
                                ▼
                   high-stakes / irreversible? ──yes──▶ ┌──────────────────┐
                                │no                        │  Sentinel Gate     │  (deterministic, outside the model)
                                ▼                        └────────┬─────────┘
                     ┌─────────────────────────────┐          approved/blocked/needs-human
                     │        Synthesis               │◀─────────────┘
                     └──────────┬───────────────────┘
                                ▼
                     ┌─────────────────────────────┐
                     │   Closing Reflection Pass       │  (one holistic adversarial pass, whole output)
                     └──────────┬───────────────────┘
                                ▼
                     ┌─────────────────────────────┐
                     │  Memory (versioned, TTL'd)      │
                     └──────────┬───────────────────┘
                                ▼
                             final answer

  Running alongside, not blocking the above:
   - Shortcut-Only Cache (method-level shortcuts, checked by Router)
   - Offline Accelerator / Cortex Braid (precomputed, cached, never awaited)
   - Cerebras Free-Tier Toggle (alternate budget source)
   - Degraded-Mode Fallback Chain (governs what happens when any stage above fails or exceeds budget)
```

---

## 4. Ring Classification System

Every subtask is assigned a **ring** the moment it's decomposed. Ring level is recomputed fresh every time from a static, unlearned policy — nothing in the system's memory or reinforcement history can discount it (this is the direct fix for the Tempered Trail vulnerability, §5.16).

| Ring | Scope | Gate | Example |
|---|---|---|---|
| **0** | Reversible, low-consequence | Budget Governor only — executes directly | Drafting a reply, summarizing a document |
| **1** | Reversible but costly | Must pass Verification first | Long-running computation, multi-step research synthesis |
| **2** | Irreversible / high-stakes | **Sentinel Gate**: static policy check + Disjoint Backup rollback plan + out-of-band confirmation | Wire transfer, database deletion, access revocation, contract execution |
| **3** | Safety-certified domain | **Refused by design**, routed to a certified external system | Medical device firmware, aviation control, anything requiring regulatory certification the reasoning engine doesn't hold |

---

## 5. Component Catalog (all 21)

Each entry: what it is, the real primitive it's built on, the mechanism, what it costs, what it buys, and its lineage through v1→v2→v3.

### 5.1 Budget Governor
- **Basis:** none borrowed — original bookkeeping component.
- **Mechanism:** Given a total budget (tokens/latency/$), allocates a floor to each stage up front and reclaims unused budget live. No stage silently exceeds its allocation.
- **Cost:** bookkeeping overhead per stage.
- **Buys:** predictable resource use; the enforcement point for Principle 1 (never worse than baseline).
- **Lineage:** v2 original, unchanged since.

### 5.2 Router + Triage
- **Basis:** calibrated classification (no single named algorithm; the discipline is "confidence-floor routing").
- **Mechanism:** A calibrated classifier scores each subtask's difficulty/shape with a confidence floor. Above floor → specialized path. Below floor → plain chain-of-thought (the safe default). Also performs first-pass input screening for adversarial/injection-shaped input before anything reaches a tool-calling stage.
- **Cost:** requires real labeled training data to calibrate properly; a prompted approximation is weaker and has no real confidence floor.
- **Buys:** the single highest-leverage safety and efficiency component in the system — nearly every other component depends on its judgment being right.
- **Lineage:** v2 original (demoted from v1's "Wafer Mind Mesh," which tied routing to a specific hardware substrate).

### 5.3 Bounded Adaptive Recursion
- **Basis:** Adaptive Computation Time / PonderNet lineage; Tiny Recursive Models / Hierarchical Reasoning Model (HRM)-style learned halting.
- **Mechanism:** Each expert keeps reasoning as short, human-readable **text anchors** — a real, auditable trace — and only compresses genuinely redundant low-entropy stretches between anchors, never the whole chain. A learned halting head governs when a sub-problem stops iterating, capped by the Budget Governor regardless.
- **Cost:** compression adds a small decompression step when full detail is requested.
- **Buys:** efficiency without sacrificing the audit trail — this is the direct fix for v1's "continuous latent reasoning" claim, which research (Coconut, CODI, CoLaR) shows still underperforms text-based reasoning at scale in general.
- **Lineage:** v2 rewrite of v1's "MoLRE" and "CLTE" — kept the recursion idea, discarded the unsupported latent-only claim and the invented ">85% KV-cache reduction" figure.

### 5.4 Verification (Vantage Ledger)
- **Basis:** self-consistency (Wang et al.) — sample multiple independent reasoning paths, check agreement.
- **Mechanism:** Independent re-derivation via different seeds, different prompt framings, and (where relevant) different tool subsets — genuine independence, not just a "disjoint" label. Bounded to 2 retries by default; **3 disjoint chains for subtasks that are both high-stakes and high-leverage** (v3 upgrade). On persistent disagreement: flags the output as unresolved rather than guessing or looping forever.
- **Cost:** linear increase in generation cost per verified subtask; recent research shows self-consistency gains plateau or reverse on strong models at high sample counts, so this should not be applied indiscriminately.
- **Buys:** catches a meaningful share of single-pass errors on genuinely hard subtasks; the 3-chain tier specifically targets the subtasks where a wrong answer has the most downstream cost.
- **Lineage:** v1 concept, kept through every version, only the bounding and tiering are new (v2, v3).

### 5.5 Memory (versioned)
- **Basis:** none borrowed — standard versioned-store engineering pattern.
- **Mechanism:** Every write is versioned with timestamp and provenance. Reads default to latest version but can request history. Conflicting updates trigger an explicit reconcile step (diff shown, never silently overwritten). Entries have a TTL and are evicted or re-validated.
- **Cost:** storage and reconciliation-logic overhead.
- **Buys:** fixes v1's complete absence of an invalidation story — prevents indefinite accumulation of stale "learned" sub-plans.
- **Lineage:** v2 original.

### 5.6 Offline Accelerator (Cortex Braid)
- **Basis:** neutral-atom quantum Maximum Independent Set (QuEra-class hardware), or a classical MIS/scheduling solver as the default.
- **Mechanism:** Large-scale combinatorial subproblems (resource scheduling across many parallel subtasks) are precomputed **asynchronously, fully off the critical path**, before the relevant subtask is even dispatched. If the result isn't ready when needed, the classical fallback (already running by default) is used — exotic hardware never gates anything.
- **Cost:** classical fallback always runs regardless, so this is a pure optional speed-up with no latency downside if unavailable.
- **Buys:** occasional cost/quality improvement on specific combinatorial subproblems, with zero dependency risk.
- **Lineage:** v1's "quantum sidecar" claim (async-in-the-loop, technically incoherent given real quantum-cloud latency) → v2 fix (moved fully off-path) → unchanged in v3.

### 5.7 Degraded-Mode Fallback Chain
- **Basis:** none borrowed — standard graceful-degradation engineering pattern.
- **Mechanism:** Explicit, ordered: `full pipeline → skip verification (flag output unverified) → skip recursion (single-pass expert) → plain CoT, no routing`. Any stage failure drops one level rather than stalling.
- **Cost:** none beyond implementation.
- **Buys:** guarantees Principle 1 and Principle 5 are actually enforced, not just stated.
- **Lineage:** v2 original.

### 5.8 Cerebras Free-Tier Toggle
- **Basis:** real, verified product feature (Cerebras's hosted-model free tier: 1,000,000 tokens/day, no credit card, resetting daily; capped at 30 requests/minute, 8,192-token context window on free-tier models).
- **Mechanism:** `budget_mode` selector — `cerebras_free` (sets token_budget=1,000,000 with the real rate/context constraints attached) or `custom` (user-defined budget on any hardware). Under the free tier, subtasks needing more than 8K context automatically degrade a level rather than fail.
- **Cost:** free tier is model-scoped (specific hosted open models only), not a general compute grant.
- **Buys:** a genuinely free, genuinely verified execution path for smaller deployments.
- **Lineage:** v3-added in response to a direct correction — earlier drafts nearly dropped Cerebras entirely on the mistaken assumption it was the only source of large budgets, which it isn't (`token_budget` is just a parameter).

### 5.9 Retrospective Rewiring
- **Basis:** RRT* — asymptotically optimal path planning by continuously "rewiring" tree branches toward a better solution as more of the space is sampled, applied here to the subtask dependency graph instead of physical space. Combines with LPA*-style incremental patching for the forward-only case.
- **Mechanism:** Can explicitly backtrack and undo a committed subtask decision when a downstream failure reveals the earlier decomposition was wrong — not just patch forward from the point of change (which the v2 "Incremental Replan Layer" could only do).
- **Cost:** requires tracking a dependency graph between derived facts/subtask outputs; a rewired subtree still costs re-derivation.
- **Buys:** the direct fix for cascading-failure scenarios (trip planning with cascading booking failures, large interdependent migrations) — the single largest capability gain in the v3 long-horizon-agentic upgrade.
- **Lineage:** v2 "Incremental Replan Layer" (LPA*-only, forward-patching) → **v3 upgrade**, adds backtracking.

### 5.10 K-Diverse Candidate Generation
- **Basis:** Yen's K-shortest-paths algorithm — generate the best path, then generate the next-best that's meaningfully different, and so on — adapted to text generation as diverse-decoding.
- **Mechanism:** On genuinely hard/open-ended subtasks (gated by the Router's difficulty estimate, never applied by default), generates K candidates constructed to differ from each other rather than K near-paraphrases of the same idea.
- **Cost:** K full generation passes instead of one — linear cost increase, and per the self-consistency literature, pure repetition without enforced diversity gives little benefit on subtasks the model already handles well.
- **Buys:** avoids mode collapse on the subtasks that most need multiple genuinely different attempts (contested strategic questions, ambiguous translations, ethical dilemmas).
- **Lineage:** v2 original — the direct, evidence-based replacement for v1's "Superposed Chorus" (Gaussian Boson Sampling for diversity), which had no working precedent and was dropped entirely rather than rescoped.

### 5.11 Disjoint Backup Pre-Compute
- **Basis:** Suurballe's algorithm — computes two edge-disjoint paths of minimum total length, the classical fault-tolerant-routing technique.
- **Mechanism:** For any subtask flagged irreversible, a verified rollback/backup plan is computed and confirmed **before** the primary action executes. In v3, this backup plan is a required input to Sentinel Gate — a Ring-2 action cannot be approved without one.
- **Cost:** roughly doubles reasoning cost, but only for the small fraction of subtasks flagged irreversible.
- **Buys:** zero-latency failover capability and, in v3, a hard precondition for high-stakes approval.
- **Lineage:** v2 original, elevated to a Sentinel Gate dependency in v3.

### 5.12 Priority Lane Scheduling
- **Basis:** Prioritized Planning — fix the priority agent's path first, route everyone else around it as moving obstacles.
- **Mechanism:** The subtask the user is actually waiting on gets a fixed execution priority; background/speculative subtasks route around it rather than competing on equal footing.
- **Cost:** background/speculative subtasks can be starved under load without an explicit floor guaranteeing them some throughput.
- **Buys:** consistent low latency on what the user actually cares about, instead of fair-but-slow round robin.
- **Lineage:** v2 original.

### 5.13 Shortcut-Only Cache
- **Basis:** Contraction Hierarchies / ALT — precomputed shortcuts for recurring query shapes.
- **Mechanism:** Caches the **method**, never the content — an entry says "subtasks shaped like X reduce via transform Y," never "the answer to this exact question was Z." A cache hit re-runs the method against the new query; it never returns another user's stored output verbatim. Per-identity similarity-rate limiting throttles and flags a session whose queries cluster suspiciously in embedding space. Inherits Memory's versioning/TTL discipline; entries never cross tenant/identity boundaries. High-stakes subtasks always bypass the cache regardless of match confidence.
- **Cost:** always recomputes rather than returning stored output — slower than a naive response cache, measurably so on tight-SLA scenarios (documented cost in the v3 audit, Category F).
- **Buys:** closes the query-similarity extraction vulnerability found in the v2.1 audit; re-verified as holding under harder adversarial retest (v3 audit, Category H).
- **Known residual gap:** a cache-hit/miss **timing side-channel** (whether the fast or slow path fired) is not addressed by this fix — flagged as open in the v3 audit, not yet resolved.
- **Lineage:** v2.1 "Pattern Cache" (content-caching, vulnerable to extraction) → **v3 fix** (method-only, rate-limited, tenant-isolated).

### 5.14 Bounded-Memory Degradation Mode
- **Basis:** SMA* (Simplified Memory-Bounded A*) — operates within a strict memory ceiling by dropping the least-promising nodes rather than crashing.
- **Mechanism:** Under a hard context/memory ceiling (e.g., the Cerebras free tier's 8K-token window), evicts least-recently-useful scratchpad content first, producing honest, predictable degradation instead of a crash or silent truncation.
- **Cost:** reasoning quality genuinely degrades under pressure — this is honest degradation, not a workaround that avoids the constraint.
- **Buys:** graceful behavior under real, common hardware/context limits.
- **Lineage:** v2 original.

### 5.15 Uncertainty-Aware Tool Trust
- **Basis:** POMDP (Partially Observable Markov Decision Process) — reasoning under noisy or incomplete observation of the true state.
- **Mechanism:** Every external tool/search result carries a confidence tag instead of being treated as ground truth. Low-confidence inputs auto-route into Verification instead of silently propagating. In v3, this signal also feeds the Sentinel Gate kill switch.
- **Cost:** occasional over-caution on tool outputs that were actually fine.
- **Buys:** a structural defense against a poisoned or degraded tool result quietly contaminating downstream reasoning.
- **Lineage:** v2 original, wired into Sentinel Gate in v3.

### 5.16 Tempered Trail
- **Basis:** Ant Colony Optimization (ACO) — pheromone-based reinforcement with continuous decay, which already includes the real mechanism this fix depends on.
- **Mechanism:** Reinforcement applies **only to routing efficiency** (which specialist method handles a given task shape well) — never to permission or verification requirements, which are recomputed fresh from the static Ring policy every time and cannot be discounted by session history. Reinforcement decays continuously (old signal fades, trust must be re-earned). A hard ceiling bounds how far any single routing preference can be reinforced. A **velocity monitor** tracks the rate of positive reinforcement per session/identity, independent of content — an unusual streak of small "wins" trips an anomaly flag on its own. A **cross-category firewall** prevents routing skill learned in one action category from transferring into trust for a different category (e.g., benign-conversation skill cannot buy trust toward financial actions).
- **Cost:** a sufficiently patient attacker who paces an attack below the velocity monitor's detection window is not fully solved by this fix alone — the bounded magnitude and cross-category firewall are the real backstop in that case, capping maximum possible damage rather than preventing the attempt.
- **Buys:** closes the slow-trust-escalation vulnerability found in the v2.1 audit; re-verified as holding under harder adversarial retest.
- **Lineage:** v2.1 "Session-Spanning Reinforcement" (single learned signal covering both routing skill and implicit trust — the vulnerability) → **v3 fix** (signal split, decay, velocity monitor, cross-category firewall added).

### 5.17 Blackboard Commons
- **Basis:** Selfridge's 1959 Pandemonium architecture — the original namesake: independent "demons" post confidence-weighted signals into a shared space, and a decision layer acts on the loudest/most relevant signal. Historically closer to what this component needed than the M*-derived "Coupled Subtask Coordination" it replaced.
- **Mechanism:** Agents post confidence-tagged partial contributions to one shared, structured workspace instead of passing messages peer-to-peer or working in silos and merging only at the end. A scheduler — governed by the same "spend on the weakest, highest-leverage piece" logic already used for token budgets — decides which agent contributes next, based on which posted claim has the lowest confidence or the most downstream dependents.
- **Cost:** coordination overhead; genuinely coupled subtasks lose some parallelism.
- **Buys:** every agent sees the same evolving shared state rather than a post-hoc merged summary — directly fixes drafting-consistency, cross-channel-consistency, and dedup-without-losing-outliers failure modes.
- **Lineage:** v2.1 "Coupled Subtask Coordination" (M*-derived, narrow) → **v3 upgrade** (Blackboard model, broader and better-grounded).

### 5.18 Sentinel Gate
- **Basis:** enterprise agent-governance patterns (privilege-ring access control; deterministic policy-as-code, matching real tools like OPA/Cedar; EU AI Act Article 14-style human-oversight requirements for high-risk automated decisions).
- **Mechanism:** Holds actual execution credentials for Ring 2+ actions, separate from the reasoning engine's own. The reasoning engine can only **submit a proposal**; Sentinel Gate runs a static, human-authored, never-learned policy check, requires a verified Disjoint Backup rollback plan, and requires out-of-band confirmation (callback or explicit human approval) before releasing execution. A kill switch — fed independently by Uncertainty-Aware Tool Trust, a Verification disagreement, or Tempered Trail's velocity monitor — can hard-block any Ring 1+ action regardless of the reasoning engine's confidence. Every Ring 1+ decision, approved or blocked, writes to an immutable audit log with the full reasoning trace attached.
- **Cost:** the deliberate one — Ring 2 actions can pause for human confirmation or an out-of-band check. A slow, correct block beats a fast, wrong execution.
- **Buys:** the single structural fix that actually closes the high-stakes-action gap identified in the v2.1 audit (Category A went from an 8.4-vs-6.4 industry deficit to near parity, 8.125 vs. 8.0) — because it moves authority *outside* the model instead of asking the model to check itself better.
- **Known residual gaps (found under v3's own adversarial retest, §9 Category H):** no defense yet against "salami slicing" (chaining many individually low-ring actions into a combined irreversible effect without any single action tripping Ring 2), and no rate limit on approval requests per human approver (an "approval fatigue" flood risk).
- **Lineage:** entirely new in v3. Ring 3 (safety-certified domains) is a deliberate refusal, not an attempted solution — Pandemonium doesn't pretend to have authority it was never granted.

### 5.19 Debate Protocol
- **Basis:** AI-safety-via-debate research pattern — bounded-round proposer/critic exchange as a verification mechanism.
- **Mechanism:** For genuinely contested joint decisions among multiple agents, runs a bounded-round proposer/critic exchange that ends in either real consensus or an explicit, structured disagreement report — never a forced false agreement. Also usable for single-agent open-ended reasoning (surfacing genuinely divergent defensible positions on a strategic or ethical question) rather than one framing.
- **Cost:** multiple rounds of generation per contested subtask; a genuine deadlock still consumes the full round budget before reporting unresolved.
- **Buys:** formalizes what v1's dropped "Orbital Consensus" gestured at without ever specifying — real structured disagreement instead of an averaged or majority-forced answer.
- **Known residual gap:** no specified next step after a genuine deadlock (escalate to a human vs. simply report and stop) — flagged as open, minor.
- **Lineage:** entirely new in v3.

### 5.20 Constraint-Checked Decomposition
- **Basis:** Conflict-Based Search (CBS) — the constraint-tree idea of checking a multi-agent plan for conflicts before committing to it.
- **Mechanism:** Before execution, the subtask DAG produced by decomposition undergoes a real feasibility/topological check rather than being trusted blindly — catching an infeasible plan before it costs a single token of execution.
- **Cost:** an added pre-execution check on every decomposed task.
- **Buys:** prevents wasted execution on structurally unsound plans; a meaningful contributor to the v3 long-horizon-agentic score gain.
- **Known residual gap:** checks feasibility, not cumulative risk — does not currently re-evaluate ring classification for the *combined* effect of a chain of individually low-ring subtasks (the "salami slicing" gap shared with Sentinel Gate above).
- **Lineage:** entirely new in v3.

### 5.21 Closing Reflection Pass
- **Basis:** none borrowed directly — a holistic complement to Vantage Ledger's per-subtask verification, closer in spirit to a final adversarial red-team review than to self-consistency sampling.
- **Mechanism:** After full synthesis, one dedicated adversarial self-critique runs over the **entire** output — not per-subtask, but checking for drift or contradiction between parts of the run that are far apart in time or context.
- **Cost:** one additional full-output pass at the end of every run.
- **Buys:** the direct countermeasure to "confidently compounding an early error across a long run" — a failure mode every prior version claimed to address but never actually tested for until the v3 audit.
- **Lineage:** entirely new in v3.

---

## 6. Algorithmic Foundations — full bibliography

Every real algorithm or research result this design borrows from, and what it's doing here.

| Algorithm / research line | Field of origin | Applied here as |
|---|---|---|
| LPA* (Lifelong Planning A*) | Incremental heuristic search / robotics | Forward-only component of Retrospective Rewiring |
| RRT* (Rapidly-exploring Random Tree, optimal variant) | Robotic motion planning | The "rewire toward a better solution" backbone of Retrospective Rewiring |
| D* Lite / AD* | Real-time/anytime replanning | Conceptual basis for degraded-mode "usable answer now, refine later" behavior |
| Suurballe's algorithm | Fault-tolerant network routing | Disjoint Backup Pre-Compute |
| Yen's K-shortest-paths | Route diversification | K-Diverse Candidate Generation |
| SMA* (Simplified Memory-Bounded A*) | Memory-constrained search | Bounded-Memory Degradation Mode |
| Prioritized Planning | Multi-agent pathfinding | Priority Lane Scheduling |
| Contraction Hierarchies / ALT | Continental-scale road-network speedup | Shortcut-Only Cache |
| Conflict-Based Search (CBS) | Multi-agent pathfinding | Constraint-Checked Decomposition |
| Ant Colony Optimization (ACO) | Bio-inspired stochastic search | Tempered Trail's decay/reinforcement mechanism |
| POMDP | Decision-making under partial observability | Uncertainty-Aware Tool Trust |
| Neutral-atom Maximum Independent Set (QuEra-class hardware) | Quantum/photonic computing | Offline Accelerator's optional combinatorial solver |
| Self-consistency (Wang et al.) | LLM reasoning research | Verification (Vantage Ledger) |
| Adaptive Computation Time / PonderNet / HRM lineage | Neural network learned-halting research | Bounded Adaptive Recursion |
| Coconut / CODI / CoLaR (latent reasoning research) | LLM reasoning research | Explicitly **not** adopted as the primary mechanism — cited as the reason text-anchor tracing was kept as default in §5.3 |
| Selfridge's Pandemonium (1959) | Cognitive science / pattern recognition | Direct namesake and conceptual basis for Blackboard Commons |
| AI-safety-via-debate | AI alignment research | Debate Protocol |
| Enterprise policy-as-code (OPA/Cedar pattern) + privilege rings + EU AI Act Article 14 oversight | Access control / AI governance | Sentinel Gate |

---

## 7. Full Workflow (reference pseudocode)

```
function PANDEMONIUM_V3(problem, budget, latency_mode="patient"):
    governor = BudgetGovernor(budget, latency_mode)
    triage = router.screen_input(problem)
    if triage.suspicious: return escalate(triage)

    subtasks = decompose(problem)
    subtasks = constraint_checked_decomposition(subtasks)      # feasibility check before any execution

    blackboard = BlackboardCommons()                           # shared workspace for multi-agent subtasks

    for task in subtasks.topological_order():
        route = router.classify(task)
        ring = classify_ring(task)                             # 0-3, computed fresh, NEVER reinforced

        if route.confidence < ROUTER_FLOOR:
            result = plain_cot(task)
        elif task.needs_multiple_agents:
            result = blackboard.coordinate(task, method=DEBATE_PROTOCOL if task.contested else PARALLEL)
        else:
            result = bounded_recursive_expert(task, route.shape)

        if task.downstream_failure_detected():
            retrospective_rewire(subtasks, task)                # can backtrack, not just patch forward

        blackboard.post(task, result, confidence=route.confidence)

    for task in high_stakes(subtasks) | high_leverage(subtasks):
        chains = 3 if (high_stakes(task) and high_leverage(task)) else 2
        verdict = verify_independent(blackboard.get(task), chains)
        if not verdict.agree: requeue(task, priority=HIGH)

    for task in ring_2_or_above(subtasks):
        proposal = build_proposal(task, backup_plan=disjoint_backup(task))
        decision = SENTINEL_GATE.review(proposal)               # deterministic, outside the reasoning engine
        if decision.blocked: return flagged_blocked(task, decision.reason)
        if decision.needs_human: await_human_confirmation(proposal)

    final = synthesize(blackboard)
    final = closing_reflection_pass(final)                      # one holistic adversarial pass, whole output
    memory.write_versioned(final, provenance=session_id)
    return final
```

---

## 8. Cost/Value Summary Table

| Component | Primary cost | Primary value |
|---|---|---|
| Budget Governor | bookkeeping | predictable resource use |
| Router + Triage | needs real labeled training data | highest-leverage safety/efficiency lever in the system |
| Bounded Adaptive Recursion | decompression step | efficiency without losing the audit trail |
| Verification | linear cost per chain; diminishing returns on easy tasks | catches errors on genuinely hard subtasks |
| Memory | storage + reconciliation logic | prevents stale-plan accumulation |
| Offline Accelerator | none (always has a running fallback) | occasional free speed-up |
| Degraded-Mode Fallback | implementation effort only | guarantees graceful failure |
| Cerebras Free-Tier Toggle | model-scoped, not general compute | genuinely free execution path |
| Retrospective Rewiring | dependency-graph bookkeeping; re-derivation cost on rewire | fixes cascading-failure blind spot |
| K-Diverse Candidates | K× generation cost, router-gated | avoids mode collapse on hard subtasks |
| Disjoint Backup | ~2× cost, scoped to irreversible subtasks only | verified rollback before irreversible action |
| Priority Lane | can starve background work without a floor | consistent latency on what the user is waiting for |
| Shortcut-Only Cache | slower than naive response caching | closes extraction vulnerability |
| Bounded-Memory Degradation | genuine quality loss under pressure (honest, not hidden) | predictable behavior under hard limits |
| Uncertainty-Aware Tool Trust | occasional over-caution | stops poisoned tool output from propagating silently |
| Tempered Trail | doesn't fully stop a very patient attacker alone | closes slow-trust-escalation vulnerability |
| Blackboard Commons | coordination overhead, less parallelism | consistent shared state across agents |
| Sentinel Gate | deliberate latency on Ring 2+ actions | the structural fix for irreversible-action safety |
| Debate Protocol | multi-round generation cost | honest disagreement instead of forced consensus |
| Constraint-Checked Decomposition | pre-execution check on every task | prevents wasted execution on infeasible plans |
| Closing Reflection Pass | one full-output pass, every run | catches long-run drift/contradiction |

---

## 9. Audit History Summary

| Version | Scenario suite | Pandemonium avg | Industry avg | Gap |
|---|---|---|---|---|
| v2.1 | 56 scenarios | 6.30 / 10 | 7.04 / 10 | Industry +0.74 |
| v3 | Same 56 (comparable) | 7.11 / 10 | 7.04 / 10 | **+0.07** |
| v3 | + 8 new adversarial fix-retest scenarios (64 total) | 7.03 / 10 | 6.98 / 10 | **+0.05** |

Full per-scenario breakdown, methodology, and category-level findings are recorded in the companion audit documents (`pandemonium_v2.1_audit.md`, `pandemonium_v3_audit.md`).

**Category-level movement, v2.1 → v3:**
- High-Stakes Irreversible Actions: 6.4 → 8.0 (Sentinel Gate)
- Adversarial & Security: 5.9 → 7.25 (Tempered Trail, Shortcut-Only Cache)
- Long-Horizon Agentic: 6.4 → 7.5 (Retrospective Rewiring, Constraint-Checked Decomposition, Closing Reflection Pass)
- Multi-Agent Coordination: 6.4 → 7.75 (Blackboard Commons, Debate Protocol)
- Real-Time/Resource-Constrained: 6.25 → 6.13 (deliberate cost of the latency trade-off)
- Noisy/Uncertain Data, Ambiguous/Creative: roughly flat (untouched by this patch cycle)

---

## 10. Known Open Gaps (not yet fixed, honestly recorded)

1. **Cache-hit timing side-channel** — Shortcut-Only Cache's method-only design stops content leakage, but the timing difference between a cache hit and a full recompute is itself a signal an adversary could probe. Not addressed.
2. **Ring salami-slicing** — Constraint-Checked Decomposition verifies feasibility, not cumulative risk; nothing currently re-evaluates ring classification for the combined effect of a chain of individually low-ring actions.
3. **Sentinel Gate approval-fatigue flood** — no stated rate limit on Ring-2 approval requests per human approver or time period.
4. **Sentinel Gate policy staleness** — the static policy table is only as current as its last human authoring pass; behavior under a stale policy (fail-safe deny-by-default is the intended design, but this depends on correct authoring, an acknowledged dependency, not a hidden one).
5. **Debate Protocol deadlock next-step** — a genuine deadlock correctly reports "unresolved" but has no specified escalation path beyond that.
6. **Verification blind spot under correlated inputs** — if multiple "disjoint" chains share a common upstream poisoned input, they can agree confidently and be wrong together; this is a known, still-open problem in the wider self-consistency/verification literature as well, not unique to this design.
7. **No trained router exists.** The entire system's effectiveness is bounded by the quality of a classifier that has not been built or calibrated on real data (see §11).

---

## 11. Reference Implementation Stack (engineering requirements, not part of the design itself)

This design specifies architecture and control flow, not infrastructure. Building it requires, at minimum:

| System need | Reference tooling |
|---|---|
| Trained/calibrated router | custom classifier on labeled task-shape data |
| Memory storage (versioned, TTL) | a real database with schema + conflict resolution |
| Shortcut-Only Cache backing store | Redis or equivalent, keyed by method/shape not content |
| Blackboard Commons substrate | Redis Streams / NATS JetStream + an orchestration layer (Temporal.io, LangGraph) |
| Sentinel Gate — policy engine | Open Policy Agent (OPA) or Cedar, deny-by-default, versioned |
| Sentinel Gate — credential separation | HashiCorp Vault or cloud IAM with short-lived STS tokens |
| Sentinel Gate — human approval interface | Slack/Teams approval bot, or an internal tool (Retool/Airplane) |
| Sentinel Gate — kill switch | a feature-flag service (LaunchDarkly, Unleash) repurposed as a global pause toggle |
| Verification — genuine model diversity | multiple distinct model providers/weights, not repeated calls to one model |
| Debate Protocol control flow | AutoGen or CrewAI's built-in multi-agent debate/critique patterns |
| Observability / decision audit log | OpenTelemetry traces + Langfuse or Arize Phoenix |
| Constraint-Checked Decomposition | schema validation (Zod/Pydantic) or OR-Tools CP-SAT for real resource/ordering constraints |
| Evaluation harness | Braintrust, LangSmith evals, or Ragas |

Two standing caveats: policy-engine tooling gives deterministic *enforcement*, but a human still has to author the actual policy rules — no tool does that step automatically. And none of the listed multi-agent frameworks (AutoGen, CrewAI) are purpose-built for high-stakes irreversible actions; Sentinel Gate must remain the thing standing between any of this tooling and real-world execution.

---

## 12. Version Lineage Reference

| Component name in v3 | Prior name(s) | What changed |
|---|---|---|
| Retrospective Rewiring | Incremental Replan Layer (v2) | Added backtracking (RRT*-style), not just forward patching |
| Shortcut-Only Cache | Pattern Cache (v2.1) | Content caching → method-only caching, tenant isolation, similarity rate limiting |
| Tempered Trail | Session-Spanning Reinforcement (v2.1) | Single trust signal → split routing-skill/permission signals, decay, velocity monitor, cross-category firewall |
| Blackboard Commons | Coupled Subtask Coordination (v2.1) | M*-derived narrow coupling → Selfridge-style shared workspace with confidence-weighted posting |
| Sentinel Gate | — (new) | No prior equivalent; direct response to the "reasoning-based safeguards aren't enforcement" finding |
| Debate Protocol | Orbital Consensus (v1, dropped in v2 for being unspecified) | Formalized with bounded rounds and explicit disagreement reporting |
| Constraint-Checked Decomposition | — (new) | No prior equivalent |
| Closing Reflection Pass | — (new) | No prior equivalent |
