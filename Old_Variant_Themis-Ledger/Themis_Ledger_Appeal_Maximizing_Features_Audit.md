# Themis Ledger — Appeal-Maximizing Features: Full Audit Before Any Suggestion

**Task framing:** brainstorm new features/mechanisms that make Themis Ledger so appealing it becomes hard to walk away from — real convenience or real efficiency, for the merchant or for the business — then run every single one through the dual-persona + `PRODUCT_CHECKPONTS.md` + `strategy_for_manipulation.txt` gauntlet already established in this project. **Nothing below the line in §3 is a suggestion unless it survived §1 in full.**

---

## 1. Full Candidate Audit

| # | Candidate | Ruthless critique | Fix / reframe | Checkpoint check | Verdict |
|---|---|---|---|---|---|
| 1 | Public "verified/sue-proof" trust badge | Structurally identical to the exact claim that got accessiBe fined $1M by the FTC — any badge implying legal safety is a compliance claim by another name, however it's worded | None survives — the risk is in the *shape* of a badge, not the wording | Fails 16, 18 outright | **REJECT, no version** |
| 2 | Auto-drafted legal response to a demand letter | Drafting response *content* to a legal demand edges into unauthorized practice of law — the same failure mode already rejected for the custody-translator idea in an earlier batch | Reframe as a data-only annex — the existing evidence export (η), repackaged and expanded, that the merchant's *own attorney* attaches — never Themis-authored legal language | 21 | **PASS, reframed — folds into η, not a new component** |
| 3 | Public accessibility "changelog" page listing every open issue | Publishing open, unresolved issues is a roadmap handed directly to a plaintiff's attorney — the opposite of protective | Publish only positive signals: "continuous monitoring since [date], N issues resolved in the past year" — never open-issue counts or detail | 21, 22 | **PASS, reframed — positive-only trust page** |
| 4 | Peer benchmark comparison ("where you rank vs. similar stores") | Could this leak competitor-identifiable data the same way μ's export could? | Runs through the *same* k-anonymity view μ already uses (N≥20) — no new data-exposure surface, just a new read of data already safely aggregated | 10, 25 | **PASS** |
| 5 | Insurance premium-discount partnership | Is this actually buildable, or is it a slide-deck fantasy dressed as a feature? | Split honestly: the *verification primitive* (an exportable, insurer-checkable proof of an active program) is buildable now, cheaply, by exposing η/β through an API. The *insurer relationship itself* is a business-development effort on a longer timeline, not a Sprint-1 engineering deliverable | 2, 8, 20 | **PASS, split into a Day-1 primitive + a Year-1 BD track** |
| 6 | Referral flywheel (mutual discount between merchants) | Does this incentivize spammy, low-quality referrals? | Cap total referral discount value per account and require the referred store to reach a paid tier before either side's discount activates — rewards a real conversion, not a click | 6 | **PASS** |
| 7 | Revenue-opportunity reframe / ROI calculator | Does this rest on real numbers or invented ones? | Verified: CDC (2024) reports 28.7% of US adults have a disability; WEF cites $13T in global disability-linked spending power; the Click-Away Pound Survey found ~71–75% of disabled users abandon an inaccessible site immediately, and separate research finds a majority would pay more for an accessible experience. All citable, all sourced, none invented | 5, 11 | **PASS, with sources attached to every number shown** |
| 8 | Pre-high-traffic-event "Surge Watch" scan | Does this need new infrastructure, or is it just smarter timing? | Pure scheduling logic on top of α — trigger an extra scan ahead of a merchant's own announced sale/launch dates, no new component | 8, 17 | **PASS** |
| 9 | WCAG-version migration readiness notice | Real risk, or manufactured urgency? | Reuses ρ's existing freshness/version tracking; only fires when a real WCAG version change actually affects the merchant's ruleset — never a manufactured deadline | 17, 25 | **PASS** |
| 10 | Jurisdiction-specific legal-deadline calendar | Risk of drifting into unlicensed legal advice, same family as #2 | Purely informational, sourced from public statute effective-dates, framed as "dates to be aware of," never "what you must do" | 21 | **PASS, with the same UPL discipline as #2** |
| 11 | Internal "statement freshness" nudge | Is this just Zeigarnik-effect manipulation? | It's an honest reflection of a real, true state (days since last verification) — the earlier reconstruction pass already established this exact distinction as acceptable when the open loop is real, not fabricated | 17 | **PASS** |

---

## 2. Rejected, and Why (grouped)

- **Compliance-adjacent public claims (badge, open-issue changelog)** fail for the same underlying reason: anything a plaintiff's attorney or the FTC could read as "this site is safe" is the one failure mode this entire product exists to avoid. No copy adjustment fixes a badge — the format itself is the problem.
- **Anything drifting into drafted legal advice** (demand-letter responses, "what you must do" legal guidance) survives only when reframed as data handed to a licensed attorney, never as Themis-authored legal content.

---

## 3. Recommended — Cleared for Suggestion

### A. Peer Benchmark Comparison
**What it is:** "Your store has fewer open critical issues than 68% of apparel stores we monitor" — computed live from μ's existing k-anonymized view (N≥20), no new data collected.
**Value:** customer convenience (instant context with zero extra effort) *and* business efficiency (zero marginal data cost — this is checkpoint 25 in action).
**Ships:** Month 1–adjacent, as soon as μ has enough contributing stores to clear the anonymity floor.

### B. Insurance Premium-Discount Program
**What it is:** an exportable, cryptographically-verifiable "active program" proof (extends η/β) that a partnered insurer's underwriting flow can check before applying a premium discount on an e-commerce liability policy.
**Value:** the single most "unavoidable" mechanism on this list — a real, external dollar saving on a bill that has nothing to do with Themis Ledger's own price, precedented by the well-established pattern of cyber-insurance discounts for demonstrated security-tool adoption (MFA, EDR), though no ADA-specific insurer relationship exists yet.
**Ships:** the verification primitive is a Day-1-adjacent, cheap build; the actual insurer partnership is a Year-1 business-development track, stated honestly as two different timelines, not one feature.

### C. Surge Watch
**What it is:** an extra α scan triggered ahead of a merchant's own announced high-traffic dates (BFCM, a product launch), catching anything a last-minute theme change introduced right when exposure and visibility both spike.
**Value:** customer convenience, delivered by scheduling logic alone — no new component.
**Ships:** Month 1, trivial addition to α's existing scheduler.

### D. Revenue-Opportunity Reframe
**What it is:** alongside the lawsuit-avoidance pitch, a second, sourced framing: CDC data putting 28.7% of US adults as having a disability, WEF's $13T disability-linked global spending figure, and abandonment/willingness-to-pay research (Click-Away Pound and related studies) showing a majority of disabled shoppers abandon inaccessible sites immediately and would spend more at an accessible one.
**Value:** widens appeal to growth-motivated merchants who don't respond to fear-based lawsuit messaging, using only cited, real numbers.
**Ships:** Week 1–adjacent — this is a copy/content addition, not an engineering build.

### E. Referral Flywheel
**What it is:** mutual discount between referring and referred merchant, activating only once the referred store reaches a paid tier — never on a click or a signup alone.
**Value:** business efficiency — directly reduces the CAC this project has repeatedly flagged as unmeasured and unaddressed.
**Ships:** Month 1, alongside the first paid tier itself.

### F. WCAG-Version Migration Readiness Notice
**What it is:** a proactive notice, sourced from ρ's own freshness tracking, the moment a WCAG version change actually affects a merchant's ruleset.
**Value:** customer convenience (ahead of the curve instead of caught off guard) and business efficiency (fewer reactive support conversations).
**Ships:** ties directly to ρ, no new component required.

### G. Positive-Only Trust Page
**What it is:** an opt-in public page stating "continuous accessibility monitoring since [date], N issues resolved in the past year" — deliberately never listing open or unresolved issues.
**Value:** customer convenience (a free trust signal for their own customers) without the litigation-roadmap risk of the rejected full-changelog version.
**Ships:** Year 1, alongside μ's benchmark report content moat.

### H. Legal-Deadline Calendar
**What it is:** jurisdiction-specific statutory effective-dates (e.g., state-level digital-accessibility law changes), framed strictly as informational awareness, never as legal instruction.
**Value:** customer convenience, low build cost.
**Ships:** Month 1–adjacent, content-only.

### I. Statement Freshness Nudge (internal)
**What it is:** an honest "last verified N days ago" indicator on δ's statement, reflecting a true state.
**Value:** minor but real nudge toward keeping the actual program current — the thing the product's entire legal value depends on.
**Ships:** trivial, alongside δ itself.

---

## 4. Roadmap Slotting

| Stage | Additions |
|---|---|
| Week 1 | D (revenue-reframe copy) |
| Month 1 | A, C, E, F, H, I |
| Year 1 | B (insurer BD track begins; verification primitive ships earlier), G |

## 5. Final Verdict

Eleven candidates audited, two rejected outright with no salvageable version, two more accepted only after a real reframe that removed their actual risk, seven cleared as proposed. The single highest-leverage idea (B, the insurance program) is also the one honestly split into "buildable now" and "not ours to promise" — which is the right way to suggest something genuinely powerful without overselling what a feature list alone can deliver.
