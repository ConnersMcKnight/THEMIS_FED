# Pandemonium v4 — Cross-System Integration Candidates

**Status:** Draft, unintegrated, pending owner review.
**Method:** Every "still honestly open after v4" item from the 7 Demons patch, checked against
Osiris Eszett, Corvus-Loopis/Corvus-Wake, CAPELLA v1/v2, and VALKRI/O2, for mechanisms that are
(a) already fully specified elsewhere in this document set, (b) close a *named* gap rather than
just sounding related, and (c) don't quietly widen the attack surface they're meant to shrink.
**Not proposed as v5.** These are candidates with dependencies and caveats attached — several are
enclave-only and would create a two-tier guarantee (self-hosted vs. SaaS), which needs a decision
from you, not a silent default.

---

## Recommended

### 1. CBAC gives Sentinel Gate the credential-separation mechanism it never specified

**Gap closed:** Sentinel Gate (v3, §5.18) says it "holds actual execution credentials for Ring 2+
actions, separate from the reasoning engine's own" — but never specifies *how*. No component
anywhere in v3/v4 addresses what happens to a delegated sub-agent's permissions when the parent
task is revoked.

**Import (from Osiris §5.2):**
- Ephemeral, cryptographically-bound, time-boxed capability tokens (not credentials the reasoning
  engine ever holds directly).
- **Epoch-Fenced Capability Validity** — a monotonic epoch counter, dispatch-time epoch-match
  check, TLA+-proven to close the issue/execute race window.
- **Sub-Agent Epoch Inheritance** — child tokens derive from the parent's epoch; parent revocation
  cascades to every descendant with zero per-child bookkeeping. This is the direct answer to the
  sub-agent-permission gap above.
- **Blast-Radius-Scoped Tombstoning** — revocation scope calibrated to detection confidence
  (low confidence → scope to the overlapping resource only; high confidence → full session kill),
  rather than Sentinel Gate's current implicit binary.

**Cost / dependency, stated plainly:** the hard sub-µs guarantee (§4/§5.2 of Osiris) only holds
when the tool gateway is enclave-local (NVIDIA H100/B200 TEE, AWS Nitro, ARM TrustZone, AMD
SEV-SNP) on self-hosted inference (vLLM/SGLang/TensorRT-LLM). On remote gateways or closed SaaS
endpoints, this degrades to a **Provisional Execution Lock** — a reversible hold backed by
Corvus-Wake's compensator pattern, not the hard guarantee. **This makes Sentinel Gate's
credential-separation strength deployment-dependent, not universal — that has to be stated in
whatever document adopts this, not left implicit.**

### 2. ARG closes a threat class Pandemonium currently doesn't watch at all

**Gap closed:** none of Router+Triage (screens *before* generation), Uncertainty-Aware Tool Trust
(screens tool *output*), or Tempered Trail (watches *session-level* reinforcement) monitor the
model's own activation trajectory *during* generation for a jailbreak/injection compromise
signature. That's a real, currently-unaddressed gap in the existing threat coverage — not one of
the seven named gaps, but worth surfacing since it's the one place this corpus already has a
built answer.

**Import (from Osiris §5.1):** Two-Tier Speculative Monitoring (cheap per-token proxy gates a full
Contrastive Linear Probe), Dual-Model Cross-Attestation (an independent second model must
corroborate before a high-confidence session kill fires — so one detector's false positive can't
unilaterally kill a session), Fail-Open Entropy Auditing (proxy-passes still get logged; an
aggregate entropy spike triggers a delayed full check even after individual passes cleared).

**Cost / dependency:** same enclave/self-hosted requirement as #1. Also — **Osiris's own §8 flags
the Contrastive Linear Probe's robustness as "asserted, not benchmarked."** Import the pipeline
*shape* (proxy → corroboration → graduated response), not a claimed detection rate. On SaaS
endpoints it degrades to output-embedding drift monitoring — weaker, explicitly not a silent no-op.

### 3. Give Cadence Demon an actual verification method for its own core claim

**Gap closed:** Cadence Demon asserts a cache hit and a full recompute are "statistically
indistinguishable from outside" (7 Demons doc). Nothing in that doc says how that claim gets
checked, or re-checked after a scheduler/OS/hardware change.

**Import (from VALKRI O2 §14):** the dudect-style Welch's-t-test harness, built for verifying
VALKRI's `select` primitive, transfers almost as-is — fixed-vs-fixed timing classes, ≥10⁶ samples,
99.9th-percentile trim, `|t| > 4.5` leak threshold, run as a **build/CI acceptance gate**, re-run
whenever the scheduler, host OS, or padding implementation changes. Swap the measured function
from VALKRI's `select()` to Cadence Demon's dispatch path; the harness code in O2 §14.2 needs
essentially no restructuring.

**Cost:** none at runtime — it's a test, not a production component. This is the lowest-risk item
on this list: it verifies an existing claim instead of adding new surface.

### 4. Audit/dry-run mode narrows Vestal Demon's one open dependency

**Gap closed:** Vestal Demon still depends on a human authoring the canary suite correctly — an
acknowledged, explicitly-not-closed dependency (both the 7 Demons doc and Sentinel Gate's original
spec say so).

**Import (from Corvus-Loopis §5.2):** the exact pattern used there for Tier-C tool-wrapping —
*"wrap the graph, replay against historical/logged traces without enforcing, emit a report before
switching to enforcing mode"* — applied to a newly authored or edited canary case: before it's
allowed to gate real deny/escalate decisions, run it in shadow mode against a window of real
recent Sentinel Gate decisions and show the author a diff ("this canary would have flipped N past
approvals to denials — confirm intended") before it goes live.

**Honest scope:** this doesn't remove the human-authoring dependency — nothing in either source
document claims it does. It moves the failure from "caught by the next scheduled canary run, after
drift already happened" to "caught at authoring time." Small, additive, no new attack surface.

### 5. Pitman-Yor/EB shrinkage prior addresses Genesis Demon's named cold-start gap

**Gap closed:** *"Genesis Demon needs real production traffic to actually mature — the cold-start
problem is solved methodologically, not instantly"* — this is the 7 Demons doc's own stated
remaining limitation.

**Import (from Osiris §5.5, #101):** Pitman-Yor process discount/concentration parameters,
MLE-fit from history, plus Empirical-Bayes shrinkage-prior pooling from parent/sibling
task-shape buckets — so a new-but-related task shape borrows calibration from its parent category
instead of starting from an uninformative default. This is explicitly **non-neural, no model
training required**, which matches Genesis Demon's own "classifier calibration, not a new model"
framing — a clean methodological fit, not a new dependency class.

**Caveat, carried over verbatim from Osiris's own limitation #7 for this same component:** helps
at the leaf/bucket level; a genuinely cold *global* deployment with no history anywhere still
starts from a noisy top-level prior, and the discount/concentration parameters need periodic
offline re-fit as traffic drifts — cheap, but ongoing, not one-shot.

**Firewall restated, not loosened:** this only touches Router+Triage's confidence weights —
Genesis Demon's existing scope. Zero access to Ring thresholds or Sentinel Gate policy, unchanged.
Importing a new calibration technique is exactly the moment scope creep is tempting; it isn't
warranted here.

### 6. Sharpen Kindred Demon's overlap score with causal-attribution isolation — narrows, doesn't close, gap 6

**Gap closed (partially — stated honestly):** gap 6 (correlated-input verification blind spot) is
explicitly flagged as irreducible in the 7 Demons doc, and that doesn't change here. What can
improve is the *precision* of Kindred Demon's existing shared-ancestry overlap score, which
currently works on whole-session lineage tags — coarse enough to both over-discount unrelated
shared context and under-discount a narrow-but-real shared poisoned span.

**Import (from Osiris #86/#87):** Causal-Attribution Isolation isolates the suspect span's actual
activation footprint via activation patching/ablation, factoring out session-context noise before
comparing chains. Dual-Channel Lineage-Gated Escalation requires *both* an exact upstream-artifact
hash match *and* the attribution-match threshold before escalating — stricter than lineage-tag
overlap alone.

**Cost / dependency:** same enclave/activation-access requirement as #1–2. Kindred Demon's current
lineage-tag mechanism is provider-agnostic and should **stay as the baseline** for SaaS
deployments; this is an optional tightening for enclave deployments, not a replacement. And to be
explicit, using Osiris's own words: this "cannot rescue a case where the poisoned source is the
only source available" — same irreducible limit, just a smaller blind spot around it.

### 7. Compose Disjoint Backup Pre-Compute with Corvus-Wake — clarification, not a new component

Sentinel Gate's Disjoint Backup Pre-Compute (Suurballe's-based) answers *"is there a redundant
path"* at planning time. Corvus-Wake's tiered compensation ledger (A/B/C tool tiers, LIFO saga
compensators, idempotency keys) answers *"how do I actually reverse what already happened"* at
execution time. These are not alternatives — Disjoint Backup Pre-Compute has no answer at all for
Tier C (witnessed, uncompensable) effects, which is the entire reason Corvus-Wake exists. Worth
stating explicitly in Sentinel Gate's own spec, since v3 doesn't currently name Corvus-Wake and a
reader of v3 alone could assume Disjoint Backup Pre-Compute is sufficient rollback coverage on its
own. It isn't.

---

## Considered, not recommended

**Unified Capability-Intent Manifold (Osiris §3, the "v88 pivot").** This is Osiris's flagship
claim — collapsing the ARG→interrupt→CBAC chain into one geometric object — but its own spec lists
the distance function `D_M` and threshold `θ` as **undefined** (§8, limitation 3), and the epoch
monotonicity proof (#84) as not yet run. Picks #1 and #2 above already give you the *pre-v88*
version (ARG detects, bridges to CBAC), which is the part that's actually specified. Don't import
the manifold fusion itself until Osiris's own Validation Priority items 1 and 4 are done — building
on an admittedly-unbuilt distance function isn't a integration risk you'd be able to see coming.

**CAPELLA's Basin Overlap Discount, for K-Diverse Candidate Generation.** Plausible surface match —
K-Diverse Candidate Generation and CAPELLA's Yen's-K-Shortest-Paths output layer share the same
late-branching pseudo-diversity risk (K outputs that look distinct but share most of their
ancestry). But CAPELLA's own v2 addendum flags this exact mechanism as *"scored without
benchmark... should not be treated as superior to DPP/QD."* If this gets imported later, it comes
with that same unproven flag attached — moving it to a new document doesn't upgrade its confidence.

**VALKRI-Turbo's adaptive fast-path techniques.** Explicitly incompatible with anything inside a
threat model per VALKRI's own §15 — the source document is emphatic that this can't be merged into
a security-relevant path without invalidating it. Not relevant to any Pandemonium component
handling Ring 1+ actions. Only a fit if Pandemonium ever needs bulk sorting/ranking over
non-sensitive telemetry, which nothing currently in scope needs.

---

## One cross-cutting note

Every "Recommended" item above except #3, #4, and #7 shares the same dependency: full strength
requires self-hosted inference on enclave hardware. If your deployment target is closed SaaS
endpoints, #1/#2/#6 still apply but at their stated degraded tier (Provisional Execution Lock /
output-embedding drift / lineage-tags-only) — worth deciding up front whether v5 documents that as
two explicit tiers (the way Osiris and Corvus-Loopis already do) rather than one spec that quietly
means different things depending on where it's deployed.
