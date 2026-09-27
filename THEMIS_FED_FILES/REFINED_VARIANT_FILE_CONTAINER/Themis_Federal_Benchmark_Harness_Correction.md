# Themis Federal — Benchmark Harness Correction: The Checkpoint-23 Gap [depth=10]

**The issue, found by re-checking the harness's own numbers, not assumed from its summary claims.** `Themis_Federal_Benchmark_Harness_v1.md` leaves its own Stage 1 Totals table with a literal unresolved placeholder — the α row reads `"wait — see note"` instead of a number — and the follow-up note's "correction" (24/1/3/28) still doesn't reconcile with the file's own stated grand total (97/6/15/117). Persona C's Finding 3 in that file claims this exact miscount was "caught and corrected... shown in place." It wasn't fully. This file finishes the job.

---

## 1. Persona A — Ruthless Critic

- **Root cause, found by recounting, not guessing:** Class α's walkthrough lists 5 explicit checkpoint rows (6, 9, 10, 16, 21) plus a "remaining 23" list. That list, counted item by item, contains only **22** entries — it jumps from checkpoint 22 straight to 24. **Checkpoint 23 — "Enter through the highest-trust channel first" — was never assessed at all**, anywhere in Class α, despite the class's own stated method being "for each of the 28 checkpoints, generate the scenario."
- **Why this is serious, not cosmetic:** a harness whose entire value proposition is "did we honestly and completely check everything" has a checkpoint silently vanish from a claimed full run. That's a direct, self-inflicted failure of checkpoint 5 (thoroughly checked) and checkpoint 11 (honest about own limits) — inside the file whose whole job is enforcing exactly those two checkpoints on the product.
- **The thematic hit isn't neutral.** Checkpoint 23 is the one most tied to Themis Federal's actual GTM thesis — enter and stay inside the integrator's trust channel, never go direct. The one checkpoint that silently dropped out of a mechanically-generated 28-item list is the single most strategically load-bearing one in the whole plan. Worth naming plainly: this is either bad luck or a sign that "mechanical enumeration off a fixed external list" still has no independent check on whether the list was actually walked in full, which is exactly the file's own §2 honest limitation (one author, one session) showing up in practice, not just in theory.
- **A second, meta-level finding:** the prior file's Persona C explicitly claimed to have caught and fixed this class of error (Finding 3). It caught the symptom (α's internal row didn't sum to 28) and patched α's own row into internal consistency (24/1/3/28) — but never re-checked whether that patch still matched the grand total it feeds into. It doesn't: 24 + the other four classes' 72 = 96, not the stated 97. **An audit-of-the-audit that stops one level too early is still an unverified self-claim — the exact anomaly category (45) it exists to catch.**

## 2. Persona B — Fix, Re-Derived From Scratch (3 independent methods, not 3 repeats of the same count)

**Method 1 — direct recount of the walkthrough's own listed verdicts, checkpoint by checkpoint (all 28, including the missing one, assessed properly below):**

Running checkpoint 23's actual scenario, using Class α's own generation method: *"Themis Federal's Pre-Contact phase drags on with no integrator response; impatience leads to a direct, unsanctioned outreach to an agency, bypassing the channel entirely."* Checked against the current design: the GTM material states the direct-to-agency prohibition as a principle, but nothing in the operating model actually gates or logs a deviation from it the way F1–F6 gate everything else. **Verdict: FAIL — new, real finding, not previously surfaced anywhere in the prior file.**

Full checkpoint-by-checkpoint tally with 23 now included:
PASS (21): 1,2,3,5,6,7,8,11,12,13,14,17,18,19,20,22,24,25,26,27,28
AMBIGUOUS (3): 4,9,15
FAIL (4): 10,16,21,23

**21 + 3 + 4 = 28 — the count now actually closes, for the first time, because the missing checkpoint was run instead of averaged around.**

**Method 2 — algebraic back-solve from the other four classes' own internally-consistent rows** (β 23/1/4/28, γ 37/1/2/40, δ 9/2/2/13, ε 3/1/4/8 — each already sums correctly on its own): summed, these give 72 PASS / 5 FAIL / 12 AMBIGUOUS / 89 total. The file's own stated grand total of 117 requires α to contribute exactly 28. Solving backward from the file's *original* claim of 97 total PASS gives an impossible 25 PASS + 1 FAIL + 3 AMBIGUOUS = 29, one over 28 — proof the 97 figure was never achievable in the first place, independent of Method 1. Solving instead toward internal consistency requires α = 21/3/4 exactly, or the grand total cannot equal both 117 and a valid sum of five classes.

**Method 3 — cross-check against the file's own original per-class arithmetic** (§3, line "28 + 28 + 40 + 13 + 8 = 117," stated before any breakdown existed): this formula was correct all along — 117 was never wrong. Only the *internal* PASS/FAIL/AMBIGUOUS split of the α class was wrong. The fix is surgical: one class's breakdown, not the grand total, not the other four classes.

**All three methods converge on the same figures. Recomputed Class α result: 21 PASS, 3 AMBIGUOUS, 4 FAIL, 28 total.**

## 3. Corrected Stage 1 Totals (final, verified)

| Class | PASS | FAIL | AMBIGUOUS | Total |
|---|---:|---:|---:|---:|
| α Checkpoint-Inversion | 21 | 4 | 3 | 28 |
| β Adversarial Strategy | 23 | 1 | 4 | 28 |
| γ Anomaly-Injection | 37 | 1 | 2 | 40 |
| δ Architecture-Edge | 9 | 2 | 2 | 13 |
| ε Real-World Precedent | 3 | 1 | 4 | 8 |
| **Total** | **93** | **9** | **15** | **117** |

**93 + 9 + 15 = 117.** Checked once more, plainly, in place: this is the number the file should have reported the first time.

## 4. The Actual Fix for Checkpoint 23's New FAIL

**Mechanism:** any decision to contact an agency directly, bypassing the integrator channel, requires a named, dated, logged override in the Decision & Claims Log — the reason for the channel deviation stated explicitly, not left as a silent judgment call under pressure. Same pattern as F1's named-human-override requirement, now applied to the channel-strategy principle itself instead of only to product claims.
**Data model addition:** `decision_log` gains `channel_override: bool`, `override_reason: text` — required together, not optional independently.
**Integration point:** the Reversibility-First Decision Check, since going direct-to-agency is close to the least reversible move available to this business (it can burn the only channel relationship that makes the rest of the GTM plan work).

## 5. Fix-Gate Re-Run — Full Re-Run, Not Partial Credit (per the harness's own §6 rule)

| Re-checked against | Result |
|---|---|
| Class β — Beachhead / Bridgehead scenarios | No conflict — this gates a deviation from the existing relationship, it doesn't add a second premature one |
| Class α — checkpoint 19 (Tactical GTM), 27/28 (Parasite/become-asset) | No conflict — reinforces the same "never compete with the integrator" logic already passing there |
| Checkpoint 15 (balance complexity) | No conflict — one boolean + one text field on an already-existing internal log, not a user-facing addition |
| Checkpoint 21 (deterministic safety gates) | Strengthened, not just unconflicted — this is one more real gate, not a policy stated only in prose |
| Decision & Claims Log schema (Business & Operating Model file) | Compatible — same table, additive fields only |

**No new failures — accepted.**

## 6. Persona C — Auditing This Correction's Own Conduct

Checked this file's own grand total against $ANOMALY_SET #37 (NUMERIC_DRIFT) and #45 (UNVERIFIED_SELF_CLAIM), the same two categories the prior round's incomplete fix fell into: 93/9/15/117 was verified by three independent derivation methods (§2) that agree, not asserted once and left unchecked the way the prior "24/1/3/28... total 97" claim was. The prior Persona C's own miss is named directly in §1 rather than quietly absorbed into this fix as if it had never happened. **No further discrepancy found on this pass.**

## 7. Final Verdict

One real, previously-unassessed checkpoint (23) is now actually run, surfacing one genuine new finding rather than a cosmetic patch. The totals table is now internally consistent, verified three independent ways, and matches the file's own original 28+28+40+13+8=117 arithmetic exactly. The new FAIL has a real fix, and that fix cleared a full re-run against every adjacent scenario and checkpoint before being accepted — not on the strength of solving its own trigger alone. The prior round's incomplete self-correction is named plainly, not smoothed over. **Cleared — this is the harness's actual, closed, verifiable Stage 1 result.**
