# Can Tribunal Become a Component? Critical Test — F7 Escalation Layer

**The question being tested, precisely:** not "is the A-B-C method useful" — that's already established across every prior file. The real question is narrower and harder: can the *same three-role structure* run as an actual mechanism inside Themis Federal's own pipeline, and does doing so solve a real, already-identified problem rather than just adding process theater to the product itself.

---

## 1. Ruthless Critique First — Why This Could Easily Be a Bad Idea

Three real objections, not straw men:

1. **Cost.** Tribunal is three model calls minimum for something a single check might resolve. Running it as a default path anywhere would be real, recurring overhead for marginal gain.
2. **The project's own governing principle cuts against this.** Every gate built so far — ν's ring ceiling, κ's whitelist, F1's allowlist — exists specifically so a high-stakes action is decided by deterministic rule, never by a model's own judgment. A Tribunal that *decides* anything risks becoming the exact thing this whole architecture was built to avoid.
3. **Fake independence.** Three roles played by one underlying model, in one session, share the same training biases by construction. Calling that "independent adjudication" would be a new instance of the `ANCHOR_BIAS` finding Persona C has already flagged twice in this project — dressed up as a fix.

Any design that survives has to answer all three, not talk around them.

## 2. The Fix That Survives — F7, an Escalation Layer, Not a Gate

**Never a default path.** F7 only runs when a component's own cheap, deterministic first-pass check returns *ambiguous* — not on every check, not as the primary mechanism anywhere. This is the same cheap-proxy-gates-an-expensive-check pattern already in this project's own corpus (Osiris Eszett's Two-Tier Speculative Monitoring), reused rather than invented, which is also the direct answer to objection 1: Tribunal's real invocation rate should be a small fraction of total volume, not the default.

**Never the gate itself.** F7 never marks a document `published`, never sets `repay_before_ship`, never resolves a claim to `verified`. It produces a routed recommendation with the full three-part record attached, logged the same way F5's debt entries are — a human confirms the actual state change, every time. This is the direct answer to objection 2.

**Honest about the third objection, not silent about it.** Stated plainly, not glossed over: an initial build of F7 running three role-prompted calls against one underlying model gets real *process* diversity (three different framings forced to engage with the same evidence) but not real *model* diversity — a systematic blind spot in the base model shows up in all three roles at once, and no prompting fixes that. Pandemonium v3's own Reference Implementation Stack already states this exact requirement for its Verification component: "multiple distinct model providers/weights, not repeated calls to one model." **The honest position: F7's single-provider version is a real, bounded improvement over no triage at all — not a claim of genuine independence.** A mature version routes Persona A, B, and C to genuinely different model providers. The cheaper version is what gets built first; the limitation is disclosed the same way every other honest limit in this project has been, not upgraded into a false claim of rigor it hasn't earned.

## 3. The Real Problem This Solves — Named, Not Invented for This Exercise

Point 12 on the project's own current open-items list is still unresolved: *solo-builder bandwidth, F1–F6 complexity vs. one person running the whole line.* Separately, points 5 and 6's fixes both routed their uncertain cases into a human review queue with no way to tell which items are urgent and which are probably fine. Combined, these are one real problem: **a single person facing an undifferentiated queue of "needs review" items will either shallow-skim all of them under time pressure, or let a backlog build and eventually rubber-stamp it under deadline pressure — which is the exact `SYCOPHANCY`-shaped failure this whole project has been checking for everywhere else.**

F7 doesn't remove the human from the loop. It triages what reaches them: auto-resolving the cases where one side of the argument shows a nameable, correctable flaw, and clearly pre-briefing the genuinely hard cases with both sides of the case already built — so a solo builder's limited attention goes to the few things that actually need it, with a head start, instead of being spread evenly across everything.

## 4. Worked Test — a Real Case, Not a Hypothetical

Running F1's cheap re-verification check against a real figure already used in this project: the "79% of 2026 accessibility lawsuit filings target e-commerce" claim.

**Cheap check result:** re-fetching the underlying source returns language close to, but not identical to, the claim — the source hedges ("high 70s percent range, per some industry trackers") rather than stating a clean, singular 79%. Not a clear match, not a clear mismatch. **Escalated to F7.**

- **Persona A:** the source doesn't actually assert a precise 79% — it's a hedged range attributed to unnamed secondary trackers. Presenting it as a clean figure overstates the source's own certainty. This is the exact shape of `FALSE_CONFIDENCE` (27).
- **Persona B:** the underlying direction is still accurate — e-commerce dominance in the filing mix is corroborated elsewhere in this project's own research. The fix is phrasing, not deletion: "roughly 79%, per one industry estimate," matching the source's own hedge instead of erasing it.
- **Persona C:** A and B aren't in substantive disagreement — both accept the underlying direction. The dispute is phrasing precision, which is mechanically resolvable, not a genuine ambiguity that needs a human's judgment on substance. **Auto-route toward B's phrasing fix, logged as a precision correction, flagged low-priority** — not silently applied, and not queued at the same urgency as a genuine, unresolved factual dispute would be.

This is the actual value demonstrated, not asserted: a naive two-call disagreement system would have logged "conflict, needs review" with no further signal. F7's third role turned that into "resolvable, low-priority, here's the specific fix" — which is a materially lighter ask on a solo builder's attention than an undifferentiated flag would have been.

## 5. Checkpoint Audit

| Checkpoint | Status |
|---|---|
| 21 — deterministic gates, not model judgment | Held — F7 recommends, a human confirms every state change |
| 15 — balance complexity | Held by the escalation-only trigger — this isn't a new default path anywhere |
| 8 — efficient | Reuses Osiris's two-tier pattern and Pandemonium's own stated model-diversity requirement rather than inventing either |
| 11 — honest about its own limits | The single-provider version's fake-independence risk is named directly, not smoothed into a stronger claim than it's earned |

## 6. One More Honest Check — Should This Be Built Now?

Tested against the project's own actual state: zero real documents published, zero real integrator-facing artifacts shipped, scaffolding-only development so far. **F7 solves a volume problem that doesn't exist yet.** Specifying it now is worthwhile — the design is sound and cheap to write down. Building it now would be exactly the premature complexity checkpoint 15 exists to catch. **Recommendation: specify F7 now (done, above), trigger its actual build the first time a real batch of F1 spot-checks or F5 auto-detected shortcuts produces a genuine backlog — not on the Day-1/Week-1 timeline, and not before.**

## 7. Final Verdict

Tribunal survives the translation from an audit method into a product component only in a specific, narrower form than "just run A-B-C on everything": escalation-only, recommendation-only, single-provider limitation disclosed rather than hidden, and deliberately not built until real volume justifies it. It solves a real, already-named problem — an undifferentiated review queue on a solo-builder timeline — rather than one invented to justify the exercise. **Passes, in this specific, bounded form — and not in any more ambitious form than this.**
