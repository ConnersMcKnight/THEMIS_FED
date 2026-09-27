# Themis Federal — Operations & Business Practices [depth=10]

**Scope of this file:** the non-code half — how the business side of Themis Federal actually runs, using only mechanisms that translate into a real practice or a real document a founder maintains. Companion to `Themis_Federal_Structural_Additions_Audit.md`, which covers the product/software side.

---

## 1. Business Practices

| Practice | What it does | When it runs |
|---|---|---|
| **Reversibility-First Decision Check** | Before a real commitment (integrator selection, pricing model, first hire), the call is framed by how reversible it is, not just its expected value — the harder-to-undo option needs a higher bar to clear. | Any named, real commitment — not routine drafting |
| **Stakeholder-Impact Check** | Names who's actually affected by a decision before it's made: agency end-users, disability-rights stakeholders, the integrator's own reputation with its client agency. | Before any integrator-facing or agency-facing commitment |
| **Pre-Mortem Risk Pass** | Assumes the channel has already failed and works backward to the likely cause, rating probability and impact honestly rather than flagging everything "high" to look thorough. | Before the first integrator pitch; re-run on any named policy-shift or Black-swan event |
| **Multi-Source Reconciliation** | Turns separate regulatory/RFI sources into one account, explicitly checking whether apparent agreement between two agencies' language is real corroboration or just two documents quoting the same origin. | Comparing DOL's RFI against VA's, or against any future third agency's |
| **Audience-Register Discipline** | Every document written for two audiences at once — an integrator's engineers and a procurement executive — is written once per register, not one blended draft trying to serve both. | Every dual-audience deliverable (RFI response, pitch deck) |
| **Competitive-Monitoring Cadence** | Weekly-cadence tracking of the accessibility-tooling market, already a standing practice for the commercial product, extended to cover the two named federal incumbents. | Inherited, unchanged, ongoing — no new mechanism built |

**GTM strategy adoptions (unchanged from prior audit, restated as active practice, not theory):**

| Strategy | Practical form here |
|---|---|
| Beachhead | One integrator partner before a second is pursued |
| Bridgehead Expansion | Firm foothold in one integrator relationship before cross-selling to a second integrator or a second agency inside the same one |
| Parasitic / Become-Their-Asset | Never sell to an agency directly — become the integrator's asset, not their competitor |
| Foot-in-the-Door | First paid pilot license precedes any renewed/expanded contract ask |
| Value-Metric Pricing (input) | Per-deployment vs. per-seat licensing tested once a real integrator conversation exists — not decided in advance |
| Social Proof | First real agency pilot cited to the next integrator conversation — real, consented cases only |
| Loss Aversion Framing | Leans on the DOL RFI's own "currently lacks a robust capability" language rather than inventing a gap |

**Excluded, permanently:** disinformation against a competing integrator, astroturfing, personnel-planting/corporate espionage, fake-urgency or fake-authority coercion, manufactured controversy, vaporware pre-announcement to freeze a competitor's or agency's decision. Illegal or dishonest, and higher-stakes here than in a commercial channel given the procurement-integrity exposure.

---

## 2. Live Documents

Three files, no more:

1. **Build & Provenance Ledger** — every acceptance-criteria set (F3) and the running debt/provenance log (F5): what's being built, what it costs, what's borrowed vs. original.
2. **Decision & Claims Log** — every real commitment (via the Reversibility-First Decision Check), dated, plus every claims-verification result (F1/F2) on anything that shipped externally. Open/deferred items — e.g., no revenue model finalized yet — are logged as explicit, dated open entries, never left as a silent absence.
3. **Integrator Pipeline** — the 3–5 candidate integrators, outreach status, feeds directly from the GTM strategy table above.

---

## 3. Operating Rule

One hard gate, not a general aspiration: **no externally-facing artifact (outreach email, RFI one-pager, pitch deck, licensing term) is finalized without the Claims Integrity Gate (F1) and Numeric Derivation Ledger (F2) having run against it in the same session it's finalized.** Narrow and attached to a concrete event — finalization — rather than an ongoing vigilance that's easy to skip under time pressure.

Milestones are named by state, not calendar, since the actual clock depends on an integrator's willingness to respond, not on the builder's schedule: **Pre-Contact → First-Contact → Pilot → Steady-State.** The original Day-1/Week-1/Month-1/Year-1 chronology from the source idea file is kept only as a loose reference point, never a commitment.

---

## 4. Final Register

| Item | Status | Form |
|---|---|---|
| F1–F6 components | Active design, not yet built | Repo/CI artifacts — see companion audit file |
| Pre-Repackaging Dependency Map | Active now | Engineering practice |
| Defect Triage, Code Review Gate, Prompt-Engineering Discipline for γ | Dormant | Triggered on first bug report / first outside contributor / first prompt revision |
| Schema/Format Conformance Checking | Dormant | Triggered on first real integrator data exchange |
| Reversibility-First Decision Check, Stakeholder-Impact Check, Pre-Mortem Pass, Multi-Source Reconciliation, Audience-Register Discipline | Active | Business practice, run on the named trigger event each |
| Competitive-Monitoring Cadence | Active | Inherited from the existing commercial product |
| GTM strategy table (7 items) | Active | Runs through the Integrator Pipeline document |
| Build & Provenance Ledger, Decision & Claims Log, Integrator Pipeline | Active | Three maintained files |
| Disinformation/astroturfing/espionage/coercion tactics | Excluded | Permanent, not situational |

## 5. Final Verdict

Three documents, six components, five engineering practices (mostly dormant), six business practices, one hard external-facing gate, milestone-based rather than calendar-based sequencing. Nothing here describes how any assistant talks or drafts — every line is either software the codebase will hold, a file a founder maintains, or a practice that runs on a named trigger. **Cleared as the operating layer for Themis Federal.**
