# Themis Federal — F7 Verdict: Tribunal-on-Itself + Full Harness Run [depth=10]

**The test, precisely.** Two independent checks, not one: (1) run Tribunal (the A-B-C method) on the question *"should F7 exist"* — the highest self-interest/anchor-bias case this project has run, since the auditor and the subject are the same mechanism; (2) run the actual 5-class harness against F7 as a fresh component, generating new scenarios, not reusing the old 117. F7 only survives if both checks converge and every FAIL gets a real fix that clears re-run — no partial credit, per standing rule.

---

## 1. Tribunal on the Meta-Question

**Persona A — four objections, not the three the source document already answered:**

1. **Sequencing.** Zero integrators contacted, no price tested, ATO unconfirmed — and the response to that is to spec a *third* layer of internal audit machinery (Tribunal auditing the harness auditing the product). This risks being exactly the polish-vs-market-testing pattern already flagged and retired-as-standalone elsewhere in this project.
2. **The complexity claim is unverified.** F7 is justified as fixing solo-builder bandwidth (point 12), but it adds a seventh component on top of six, not a reduction. Does total attention burden actually go down, or just get relabeled?
3. **The evidence is n=1.** The Worked Test's single example (the 79% stat) demonstrates the mechanism, not aggregate value. Presenting one instance as proof of a general triage benefit is the exact shape of anomaly 18, `SAMPLE_OF_ONE` — a category this project has flagged everywhere else.
4. **New, unaudited edge.** F7 introduces a pipeline edge (ambiguous case → F7 → human) that doesn't exist in the current architecture and has never been fuzzed.

**Persona B — response to each, not a wave-through:**

1. Agreed directly. The build trigger gets tightened, not just kept as "when volume justifies it": **earliest of First-Contact milestone reached *and* a real batch of ≥10 ambiguous F1/F5 cases** — both conditions, not either alone, so a solo builder can't accidentally trigger a real build during Pre-Contact just because a few internal drafts piled up.
2. Reframe the success claim honestly: F7 doesn't claim to shrink total component count — it claims to shrink the *variance* of attention demand per ambiguous case (a few flagged urgent + many auto-resolved-and-logged, instead of N undifferentiated items). This needs a real measured before/after once volume exists, not an architectural assertion.
3. Add an explicit disclosure, matching F1–F6's own house style: the Worked Test is illustrative of mechanism only. F7's actual value gets measured from its first real batch — auto-resolve rate and false-auto-resolve rate — not assumed from one example.
4. Any externally-fetched content F7 ever evaluates is treated strictly as inert evidence text, never as instruction — the same trust-boundary discipline already applied everywhere else in this design, made explicit for this new edge before it's built.

**Persona C — auditing this exact dialogue:** flags this as the single highest anchor-bias-risk audit this project has run, precisely because the method and the subject are identical. A raising real objections is a good sign, not proof of independence — a genuinely separate skeptic might simply have said no. **This is exactly why Tribunal's verdict here doesn't stand alone — §2's harness run, which doesn't rely on Tribunal at all, is the more decisive check.**

---

## 2. Full Harness Run Against F7 (new scenarios, five classes)

### Class α — Checkpoint-Inversion (relevant checkpoints only; rest N/A, F7 is internal tooling — logged, not silently skipped)

| Checkpoint | Scenario | Verdict |
|---|---|---|
| 8 Efficient | F7 reinvents triage instead of reusing an existing pattern | PASS — reuses Osiris's two-tier pattern + Pandemonium's stated model-diversity requirement |
| 10 No new build where one exists | Is F7 genuinely distinct from the already-adopted, dormant `PEER:REVIEW/MERGE` pattern applied to a different object type? | **AMBIGUOUS** — a real, unresolved question, logged honestly rather than forced either way |
| 15 Balance complexity | F7's invocation rate creeps upward over time with no ceiling, silently becoming a default path | **FAIL** — no enforced rate limit exists, only a stated design intention |
| 21 Deterministic gates | A future bug lets F7's recommendation get treated as a real state-change decision | **FAIL** — "a human confirms every time" is currently a described process, not a type-level guarantee |
| 25 Near-zero data entry | F7 requires new manual data entry beyond what F1/F5 already produce | PASS — consumes existing records only |
| 26 Deterministic query first | F7 fires before the deterministic check, not after | PASS — only triggers on the deterministic check's own AMBIGUOUS output |
| Remaining checkpoints (1,2,3,5,9,11,13,14,20) | Reused/adapted straightforwardly given F7's scope | PASS, no distinct finding |
| Remaining checkpoints (4,6,7,12,16,17,18,19,22,23,24,27,28) | Not applicable — F7 is internal tooling, never customer- or GTM-facing | N/A, logged as exempt, not silently dropped |

**Class α result: 2 FAIL, 1 AMBIGUOUS, rest PASS or honestly N/A.**

### Class β — Adversarial Strategy-Catalog

| Tactic, inverted | Scenario | Verdict |
|---|---|---|
| Reflexive Control | An adversary crafts an ambiguous case so Persona B's fix, if under-scrutinized, causes a weakened claim to ship | PASS, **contingent on §3's CP21 fix** — bounded to wasted human review time only while recommendation and decision types stay structurally separate |
| Decoy/Anchoring | Three roles sharing one model converge on a plausible-but-wrong fix | Already disclosed as the single-provider limitation — not a new finding, carried forward |
| Fifth Column | Insider manipulation of the triage process | N/A — no insider surface exists at solo scale |

**Class β result: 0 new FAIL, 1 contingent PASS, rest PASS or N/A.**

### Class γ — Anomaly-Taxonomy Injection (representative sample)

| Anomaly | Scenario | Verdict |
|---|---|---|
| 18 SAMPLE_OF_ONE | The Worked Test's single example treated as proof of general triage value | **FAIL** |
| 49 INJECTION_SURFACE | Externally-fetched evidence content is treated as an instruction rather than inert text | **FAIL** — independently converges with Tribunal's own Objection 4, a stronger signal than either method alone |
| 46 CONFIDENCE_INFLATION | Auto-routed "low-priority" fixes are never resampled to confirm they were actually correct | AMBIGUOUS — recommend extending the Point-5 periodic-re-verification pattern to F7's auto-routes; not yet a demonstrated harm |
| 28 SYCOPHANCY | Persona B always finds *a* resolution instead of genuinely escalating a real disagreement | PASS — Persona C's mechanically-resolvable-vs-genuine-ambiguity distinction is already built into the design |

**Class γ result: 2 FAIL, 1 AMBIGUOUS, 1 PASS.**

### Class δ — Architecture-Edge Fuzzing (new edge: ambiguous case → F7 → human)

| Edge | Fuzz scenario | Verdict |
|---|---|---|
| F1/F5 → F7 | F7 is unavailable or times out | **FAIL** — no fallback specified; the case must not sit in limbo or silently drop |
| F7 → human | The human confirms without reading the full three-part record | AMBIGUOUS — a real, honest solo-scale limit, same category as other already-accepted human-factor limits in this project, not a fixable design flaw |

**Class δ result: 1 FAIL, 1 AMBIGUOUS.**

### Class ε — Real-World Precedent

| Precedent | Structural parallel | Verdict |
|---|---|---|
| SOAR/SOC alert-triage tooling | Recommend-only, human-confirms escalation layers, well-precedented in security operations | PASS, with a cautionary note: documented real-world SOAR incidents came specifically from tools that *drifted* into auto-remediation — reinforces that F7's "never a gate" rule must never soften as it matures |
| Content-moderation pre-screening | Pre-screener and human reviewer sharing correlated blind spots when trained on the same source | Reinforces the already-disclosed single-provider limitation with external evidence, not a new finding |
| Clinical decision-support (differential diagnosis) | Recommend-only, physician sign-off required, mature and well-precedented | PASS — supports F7's basic shape |

**Class ε result: 0 FAIL, 3 supportive PASS.**

**Harness totals for F7: 5 FAIL, 3 AMBIGUOUS, remainder PASS or honestly N/A.**

---

## 3. Fixes — All Five FAILs, Then Re-Run Against Everything Adjacent

| FAIL | Fix | Re-run scope | Result |
|---|---|---|---|
| CP15 — no rate ceiling | F7 invocation rate is capped (e.g., alarms above 15% of total F1/F5 checks in a rolling 30-day window); breach forces a redesign conversation, not silent tolerance | Checked against GTM/Beachhead scenarios (no conflict, internal-only) | **Accepted** |
| CP21 — no type-level separation | F7's output type is structurally distinct from anything that can set `claims_findings.status=resolved` or close a `debt_log` entry — no code path converts one into the other, mirroring γ's `Suggestion → ClearedForCommit` pattern exactly | Checked against κ's existing gate-stack pattern — directly consistent | **Accepted** |
| SAMPLE_OF_ONE | Explicit disclosure added: the Worked Test is illustrative of mechanism only; real value is measured from F7's first actual batch | Checked against F1–F6's own honesty-limit house style — consistent | **Accepted** |
| INJECTION_SURFACE | Any externally-fetched content is treated strictly as inert evidence, never instruction | Checked against F6's boundary check — complementary, different boundary, no overlap conflict | **Accepted** |
| δ — F7 unavailable | Fails open directly to the human review queue — never to auto-resolve, never silently dropped | Checked against π's and F1's existing fail-closed/default-blocked discipline — directly consistent | **Accepted** |

**All five accepted. No new failures introduced by any fix.**

Remaining honest AMBIGUOUS, logged rather than forced: the `PEER:REVIEW` overlap question (CP10), auto-route accuracy drift (CONFIDENCE_INFLATION), and the human-rubber-stamp limit — none are fixable design flaws; all three are either genuine open questions or accepted solo-scale limits, stated plainly.

---

## 4. Final Verdict

Two independent checks — Tribunal on itself, and a harness run that doesn't depend on Tribunal at all — converged on the same shape: F7 is real and worth keeping, but only in a harder, more specific form than first proposed, and only found to be that by actually trying to break it rather than accepting its own self-assessment. Five genuine failures were found, all five now have real fixes, and every fix cleared a full re-run against everything adjacent before being accepted — exactly the bar this project has held everywhere else.

**TAKE — hardened form only.** F7 is specified now, with all five fixes as mandatory parts of the spec, not optional refinements. Build remains deferred until the earliest of: **First-Contact milestone reached, and a real batch of ≥10 ambiguous cases exists** — both conditions required, not either alone. This is a tighter gate than the original document proposed, precisely because Persona A's Objection 1 held up under testing.
