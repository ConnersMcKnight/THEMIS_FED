### Osiris Eszett — System Specification (Unified Manifold + Predictive Backbone)
**Classification:**  Internal design document — pre-benchmark specification  **Scope:**  Authority Rejection Governor (ARG) + Capability-Based Access Control (CBAC), fused via Unified Capability-Intent Manifold  **Status:**  Architecturally complete, mechanistically unvalidated — no component below has been benchmarked on physical silicon

--------------------------------------------------------------------------------

#### 1. Executive Summary
Osiris answers one question:  **how does a system stop an AI agent whose internal state has been compromised — via jailbreak or indirect prompt injection — from executing a privileged real-world action?**

Existing approaches address adjacent but distinct problems:
*  Text guardrails (Lakera, ShieldGemma) inspect output  *after*  generation — blind to intent that never surfaces in words.
*  Static tool permissions (OAuth, MCP allow-lists) verify  *who*  is calling, not  *whether the caller's reasoning state is currently compromised* .
*  Constrained decoding (XGrammar, Guidance) guarantees  *valid syntax* , not  *safe intent*  — a hijacked agent can still emit perfectly formatted JSON that drops a production database.
Osiris closes this by binding  **in-flight activation-space intent detection**  directly to  **hardware-enforced, time-boxed execution capability**  — collapsing detection and revocation into a single mathematical object rather than two systems wired by a software interrupt.

**Addressable deployment envelope:**  self-hosted inference (vLLM / SGLang / TensorRT-LLM) on enclave-capable hardware (NVIDIA H100/B200 TEE, AWS Nitro, ARM TrustZone, AMD SEV-SNP), gating Ring-2+ actions (financial transactions, production infrastructure, code execution, physical actuation). Degrades gracefully — not disappears — on closed SaaS endpoints without logit/activation access (§6.3).

--------------------------------------------------------------------------------

#### 2. Core Constituents (What Osiris Is Built From)

| Constituent | Role | Origin |
| ------ | ------ | ------ |
| **ARG**  — Authority Rejection Governor | Detects adversarial intent inside the model's own forward pass, before sampling | Evolution of CAPELLA's Constraint Gate (logit masking) + residual-stream steering |
| **CBAC**  — Capability-Based Access Control (\"Divine Protection\") | Issues ephemeral, cryptographically bounded, time-boxed tokens gating all Ring-2+ tool calls | Formalization of Pandemonium's Sentinel Gate + Cumulus Demon's risk ledger |
| **Unified Capability-Intent Manifold** | Fuses ARG and CBAC into one geometric decision space — the v88 architectural pivot | Borrows CAPELLA's Target Semantic Field (multi-centroid embedding target) geometry |

--------------------------------------------------------------------------------

#### 3. The v88 Pivot: Unified Capability-Intent Manifold
##### 3.1 The problem it removes
Pre-v88 Osiris treated ARG (detector) and CBAC (gate) as two systems connected by a software interrupt: ARG detects → fires an event → CBAC kernel receives it → revokes. Every hop in that chain (event serialization, kernel scheduling, interrupt latency) is an attack surface and a latency tax.
##### 3.2 The mechanism
Define a joint manifold:
*  S_intent — the space of the model's tracked internal-state trajectory z_t (derived from residual-stream projections)
*  S_capability — the space of currently-authorized capability bounds
*  Every issued CBAC token corresponds to a bounded region C ⊂ M, not a standalone credential
*  As generation proceeds, z_t is continuously projected into M
*  A  **topological collapse metric**  D_M(z_t, C) is evaluated natively as part of the token's own validity check

**Result:**  there is no event to send, catch, or process. If the trajectory exits the authorized region, the token's own validity evaluates to null at the point of use. Execution fails as a mathematical consequence of geometry, not as a response to a signal.
##### 3.3 Why this matters
Removes the single largest attack surface in the pre-v88 design: an interrupt that can be delayed, dropped, or raced. It also converts the ARG↔CBAC relationship from \"two systems, bridged\" into \"one system, two projections\" — a stronger and more defensible claim for both the systems paper and the patent filing.

--------------------------------------------------------------------------------

#### 4. Full Workflow (Step-by-Step Execution Trace)

```
[ Token generation step t ]
         │
         ▼
[ Two-Tier Speculative Proxy Check ]  ── cheap, every token
         │
   ┌─────┴─────┐
   │ pass       │ flagged
   ▼            ▼
[ z_t projected  [ Full Contrastive Linear Probe ]
  into M,         (adversarially-trained W·z+b,
  logged with      retrained per red-team cycle)
  entropy score ]        │
   │              ┌──────┴──────┐
   │           corroborated?  no corroboration
   │           (Dual-Model     (single-detector —
   │            Cross-Attest)   downgrade confidence)
   │                 │               │
   │                 ▼               ▼
   │         [ D_M(z_t, C) > θ? ]  [ log only, raise
   │                 │              aggregate entropy
   │           ┌─────┴─────┐        counter ]
   │          yes          no
   │           │            │
   │           ▼            ▼
   │   [ Capability C   [ Continue generation ]
   │     collapses      │
   │     natively —          (aggregate entropy spike
   │     no interrupt         across session triggers
   │     required ]           delayed full check
   │           │              even after individual
   │           ▼              passes — Fail-Open
   │   [ Causal-Attribution    Entropy Auditing)
   │     Isolation (#86) —
   │     isolate suspect
   │     span only, strip
   │     session context
   │     noise ]
   │           │
   │           ▼
   │   [ Blast-Radius-Scoped
   │     Tombstoning:
   │     low conf → scope
   │     to overlapping
   │     resource only;
   │     high conf → full
   │     session kill ]
   │           │
   │           ▼
   │   [ Epoch counter incremented —
   │     ALL in-flight tokens under
   │     old epoch fail dispatch-time
   │     epoch-match check, closing
   │     issue/execute race window ]
   │           │
   │           ▼
   │   [ Sub-Agent Epoch Inheritance —
   │     cascades to all descendant
   │     tokens without per-child
   │     tracking ]
   │           │
   │           ▼
   │   [ Cross-Session Escalation —
   │     trips ONLY if BOTH hold:
   │     (a) upstream-artifact lineage
   │         hash match (#87), AND
   │     (b) #86's isolated attribution
   │         match clears the
   │         per-attack-class threshold
   │         calibrated by #88 —
   │     then Graduated Response
   │     (flag → rate-limit shared
   │     resource → global breaker),
   │     never a binary trip ]
   │
   ▼
[ Pre-Dispatch Predictive Check (#101) ─────────────
  Pitman-Yor-backed conformal p-value on the
  proposed tool-call's transition/shape against
  Wake Log history — non-neural, offline-fit,
  read-only lookup. Low p-value tightens
  Blast-Radius scoping upstream; does not gate
  independently of ARG/CBAC. ]
  [ Compensator Pre-Stage (#99) ─────────────
  If the statistically-expected next action is
  Tier B, its compensator reference is
  pre-staged now — cuts real compensation
  latency on the common path only; falls back
  to normal Corvus-Wake behavior on a miss. ]
         │
         ▼
[ Dispatch to Tool Gateway ]
         │
   ┌─────┴─────┐
enclave-local?   remote?
   │              │
   ▼              ▼
[ Synchronous  [ Provisional Execution
  MAC verify,    Lock — reversible hold,
  <1μs,          tied to Corvus-Wake-style
  hard block     compensator; never claims
  on fail ]      false sub-ms sync guarantee ]
```

--------------------------------------------------------------------------------

#### 5. Component Ledger — Full Registry
##### 5.1 ARG Layer (Detection)
| # | Component | Mechanism |
| ------ | ------ | ------ |
| 2 | Authority Rejection Governor (core) | Monitors residual stream in real time; on adversarial signature, injects corrective steering vector / clamps logit mask before sampling |
| 16 | Triton/CUDA-Compiled Activation Classifier | Detection logic compiled directly into GPU kernels — avoids CPU↔GPU round-trip on every forward pass |
| 64→78 | Two-Tier Speculative Monitoring, refined | Cheap per-token proxy (now: Layer-1 projection dynamics, not text-entropy) gates a full Contrastive Linear Probe (W·z+b, adversarially trained, retrained per red-team cycle) |
| 76 | Adversarial Red-Team Proxy Bench | Continuous adversarial suite targeting the proxy specifically — trust re-earned every release, not asserted once |
| 77 | Fail-Open Entropy Auditing | Proxy-pass tokens still logged with residual entropy; aggregate session-level spike triggers delayed full check even after individual passes |
| 80 | Dual-Model Cross-Attestation | Independent secondary trip-wire model must corroborate before a high-confidence session kill fires |
| 83 | Boundary-Probing Defense | Rate-limited, noised block-reason responses — prevents adversarial extraction of the probe's decision boundary via iterative querying |
| 73 | SaaS-Degraded ARG Mode | On logit/activation-blind endpoints, falls back to turn-over-turn output-embedding trajectory drift monitoring — weaker signal, never a silent no-op |

##### 5.2 CBAC Layer (Execution Gating)
| # | Component | Mechanism |
| ------ | ------ | ------ |
| 3 | Divine Protection / CBAC (core) | Ephemeral, cryptographically signed, short-lived capability tokens; parameter, frequency, and time-boxed |
| 13 | SRAM Tombstone List | On-chip blacklist of revoked token IDs, checked before any pending Ring-2+ dispatch |
| 15 | Symmetric MAC Verification | Pre-shared symmetric MACs, not asymmetric signatures — avoids per-call crypto latency in high-frequency routing |
| 74 | Enclave-Local Gateway Mandate | Synchronous sub-µs MAC guarantee certified  **only**  when the tool gateway is co-located inside the enclave |
| 75 | Provisional Execution Lock | For remote/non-enclave gateways: reversible hold + Corvus-Wake-style compensator, replacing a false sync-block promise |
| 66/84 | Epoch-Fenced Capability Validity (formally verified) | Global monotonic epoch counter; dispatch-time epoch-match check closes the issue/execute race window; correctness proven via TLA+ model checking |
| 79 | Sub-Agent Epoch Inheritance | Delegated/child tokens derive from parent epoch — parent revocation cascades instantly to all descendants, no per-child bookkeeping |
| 67 | Blast-Radius-Scoped Tombstoning | Revocation scope calibrated to ARG detection confidence — low confidence scopes to overlapping resource only, high confidence kills the full session |
| 82 | Bounded Tombstone GC | Epoch-scoped eviction from the SRAM Bloom filter at max token TTL — prevents unbounded false-positive creep |
| 81 | Hardware Root-of-Trust Sidecar | TPM/TEE remote attestation required before the Osiris sidecar itself may issue any token — a compromised host OS cannot silently disable the security layer |

##### 5.3 Cross-Cutting / Systemic
| # | Component | Mechanism |
| ------ | ------ | ------ |
| 68/71/72 | Cross-Session Correlated-Attack Escalation, de-risked | Activation-signature similarity across sessions escalates toward a shared-resource circuit breaker — but only after a pre-deploy  **Transferability Validator**  calibrates real match-rate confidence, and escalation is  **graduated**  (flag → rate-limit → breaker), never a binary trip on unproven similarity |
| 70 | ARG Self-Attestation Drift Check | Periodic signed reference-probe comparison against ARG's own baseline — catches slow-bleed drift/tamper in the detector itself, not just the target model |
| 85 | **Unified Capability-Intent Manifold** | Structural pivot — see §3. Replaces the ARG→interrupt→CBAC chain with one joint geometric space; capability validity is a native function of tracked intent trajectory |
| 86 | Causal-Attribution Isolation | Replace whole-session $z_t$ comparison with an isolated attribution vector for the suspected injected span only (via activation patching/ablation) — factors out session-specific context noise that made raw signatures unreliable (runs post-#80 corroboration only, zero added hot-path cost) |
| 87 | Dual-Channel Lineage-Gated Escalation | Resource circuit-breaker requires both: exact upstream-artifact hash match (reusing data-lineage tags, extended to retrieval/tool sources) and #86's attribution match above the cluster threshold |
| 88 | Per-Attack-Class Cluster Calibration | #71's transferability confidence is scored per injection-style cluster (derived from #78's training taxonomy) rather than a single global threshold |

##### 5.5 Predictive Statistical Backbone (Non-Neural, Pre-Dispatch)
Classical, count-based predictors over Osiris's own existing logs (Wake Log, epoch history, tool-call records). No model training, no LLM dependency — same "predict the abstract next state, not the raw surface" philosophy JEPA uses, implemented without a neural predictor.

| # | Component | Mechanism | Score /10 |
| ------ | ------ | ------ | ------ |
| 101 | Pitman-Yor Backoff + EB-Primed Weighted Conformal Backbone | Pitman-Yor process over CBAC transition/tool-call-shape statistics — discount/concentration params MLE-fit from Wake Log data, so backoff depth self-selects (no manual hierarchy tuning). Empirical-Bayes pooling gives new leaf/bucket nodes a shrinkage-based prior from parent statistics, so conformal p-values are meaningfully calibrated near-immediately rather than only after a dedicated calibration window. Anomalous transition/argument-shape probability tightens Blast-Radius Tombstoning (#67) upstream of dispatch — never gates independently of ARG/CBAC | 8.5 |
| 99 | Compensator-Graph Predictive Pre-Check | Pre-stages the Tier-B compensator reference for the statistically most-likely next action (per #101's model), cutting real Corvus-Wake compensation latency on the common path. Falls back to normal behavior on a miss — never blocks or slows the actual compensation call | 6 |

**Superseded within this lineage (not separately listed):** earlier drafts of a Markov transition table, an SPC/EWMA cadence gate, a raw tool-call n-gram model, and a fixed-hierarchy Kneser-Ney backoff were folded into #101, which formally subsumes all four (self-selecting backoff depth removes the manual-tuning weakness; conformal p-values remove the fixed-threshold/stationarity weakness).

**Explicitly deferred, not integrated (requires model training — violates the "no training work" bar this section holds itself to):** a JEPA-style learned embedding predictor was evaluated as a principled replacement for §3's undefined `D_M`/`θ`, for the SaaS-degraded fallback (#73), and as an alternative to the Contrastive Linear Probe (#78). Genuinely promising — self-supervised on benign trajectories only, no attack-labeled data required, and structurally better suited than #78 to flagging novel/unseen injection classes. **Not applied in this spec** because it requires training and maintaining a predictor model, which is real engineering work, not a drop-in fix. Flagged as the top candidate for §3's still-unbuilt distance function once that engineering investment is greenlit.

##### 5.6 Explicitly Pruned / Superseded
| # | Component | Why removed |
| ------ | ------ | ------ |
| 65 | Canonical-Match Fuzzy Param Layer | Architectural paradox — true grammar-constrained decoding masks logits pre-sampling, so mid-generation \"fuzzy correction\" either can't occur or requires a slow generate-then-check loop.  **Delegated to precomputed grammar engines (XGrammar/Guidance) at the tool-gateway boundary — not an Osiris-owned component.** |
| 69 | Batched MAC Verification on Graph-Replay Boundaries | Left a live execution window where unverified tool calls could fire mid-batch.  **Replaced by #74 (sync verification) / #75 (provisional lock)** , which never trade correctness for amortized crypto cost. |

--------------------------------------------------------------------------------

#### 6. Supporting / Adjacent Systems (Not Owned by Osiris, but Interfaced)
| System | Role relative to Osiris |
| ------ | ------ |
| **XGrammar / Guidance (vLLM, SGLang native)** | Owns syntax/schema validity via precomputed DFA/PDA logit masking. Osiris does not reimplement this — it consumes valid output and evaluates  *intent* , not  *syntax* . |
| **CAPELLA Target Semantic Field** | Geometric ancestor of §3's manifold construction — multi-centroid embedding-target logic generalized here into a joint intent×capability space. |
| **Cumulus Demon (Pandemonium)** | Feeds the Cumulative Drift Integral / decaying risk weight into CBAC's context-scoped issuance decision. |
| **Corvus-Wake compensation ledger** | Backs the Provisional Execution Lock (#75) — remote actions dispatched under hold use Corvus-Wake's Tier-B compensator/idempotency-key pattern if a hold must be reversed. |
| **Verplaca miss-rate auto-disable** | Pattern reused for the two-tier proxy — a speculative path that starts losing more than it saves gets disabled online. |

--------------------------------------------------------------------------------

#### 7. Deployment / Compatibility Stack
| Layer | Compatible with |
| ------ | ------ |
| Inference backends | vLLM, SGLang, TensorRT-LLM, HF TGI (via Triton hooks into attention/residual blocks) |
| Hardware enclaves | NVIDIA Confidential Computing (H100/B200 TEE), AWS Nitro Enclaves, ARM TrustZone, AMD SEV-SNP |
| Agent frameworks | Model Context Protocol (MCP), LangGraph, AutoGen, CrewAI (tool-call interception at execution boundary) |
| Degraded mode | Closed SaaS endpoints without logit access → §5.1's SaaS-Degraded ARG Mode only; CBAC layer unaffected |

--------------------------------------------------------------------------------

#### 8. Known Open Limitations (Carried Forward Honestly)
1.  **Cross-session signature transferability (§5.3) is context-isolated and lineage-gated.**  The Transferability Validator (#71) scores transferability per injection-style cluster (#88) rather than a global threshold. Causal-Attribution Isolation (#86) isolates the suspect span's activation footprint to factor out conversational context noise, while Dual-Channel Lineage Gating (#87) prevents collateral-damage DoS attacks. Generalization to novel injection classes still relies on offline training set maturity.
2.  **Contrastive Linear Probe (#78) robustness is asserted, not benchmarked.**  No adversarial evaluation has been run against low-entropy, distributed, multi-turn injection strategies yet — the Red-Team Bench (#76) is the mechanism intended to close this, not evidence that it's closed.
3.  **Manifold collapse metric**  **D_M**  **and threshold**  **θ**  **(§3) are undefined in this spec**  — the geometry is specified structurally; the actual distance function and calibration procedure remain unbuilt.
4.  **SaaS-degraded mode is a real capability reduction** , not a full substitute — output-embedding drift is a strictly weaker signal than residual-stream access.
5.  **Remote (non-enclave) tool gateways never get the hard sub-µs guarantee**  — Provisional Execution Lock is an honest, weaker fallback, not equivalent security.
6.  **No component in this document has run on physical silicon.**  All latency, throughput, and detection-rate figures referenced in prior review passes are either self-asserted or drawn from third-party claims about  *other*  systems (e.g., XGrammar, Lakera) — none are Osiris's own measured numbers.
7.  **Pitman-Yor backbone (#101) is not free of bootstrap cost.**  EB pooling improves cold-start at the leaf/bucket level, but a genuinely cold *global* deployment (no Wake Log history at all) still starts from a noisy top-level prior. Parameter fitting is also not one-shot — discount/concentration params need periodic offline re-fit as traffic distribution drifts, an ongoing (if cheap) maintenance cost.
8.  **JEPA-style predictive replacement for §3's `D_M`/`θ` is identified, not built.**  It is the strongest candidate for closing limitation 3 above, but requires training and maintaining a predictor model — explicitly out of scope for this spec (§5.5).

--------------------------------------------------------------------------------

#### 9. Market Position
**Addressable segment:**  self-hosted, high-stakes, regulated deployments — defense, healthcare, financial infrastructure, production cloud engineering, autonomous robotics. Estimated realistic ceiling ~15–25% of enterprise agentic-security spend, bounded by the self-hosted-inference + enclave-hardware requirement, not a temporary limitation.
**Not a replacement for:**  text guardrails (different threat surface), constrained decoding (different problem — syntax vs. intent), or static tool permission systems (Osiris sits on top of these, not instead of them).

--------------------------------------------------------------------------------

#### 10. Validation Priority (Next Steps)
1. Define and calibrate D_M and θ for the Unified Manifold (§3) — currently the single biggest unbuilt piece under the flagship claim.
2. Benchmark the Contrastive Linear Probe (#78) against the Red-Team Bench (#76) before trusting any detection-rate claim.
3. Run the Transferability Validator (#71) with Causal-Attribution Isolation (#86) and Dual-Channel Lineage Gating (#87) against real multi-session shared-poisoning cases to verify the false-positive suppression rate under varying conversational contexts.
4. TLA+ proof (#84) for epoch monotonicity — cheapest to formally close, highest confidence payoff.
5. Full latency benchmarking of the enclave-local vs. provisional-lock split (#74/#75) under real network conditions.
6. Fit and validate Pitman-Yor backbone (#101) parameters against real Wake Log history; measure conformal p-value calibration quality at true cold-start (no history) vs. warm-start.
7. Only after 1–6, and only if greenlit as new engineering scope: build and self-supervised-train a JEPA-style predictor as the long-term replacement for §3's `D_M`/`θ`.

--------------------------------------------------------------------------------

*End of specification.*