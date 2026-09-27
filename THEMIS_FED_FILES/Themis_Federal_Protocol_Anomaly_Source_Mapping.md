# Themis Federal — Protocol & Anomaly Source Mapping [depth=10]

**Purpose:** for every addon in the two prior files, trace it back to the exact command(s)/category(ies) in the three source files — `PROTOCOL_STANDARD_LIBRARY.md`, `Protocol_Library_MASTER_CHAIN_REFERENCE.md`, `ANOMALY_SET_Expanded (1).md` — and show the transformation logic, not just the name. Where a design element came from somewhere else entirely (Themis Ledger's own existing components, or nowhere in these three files at all), that's stated explicitly rather than folded in as if it were sourced the same way.

---

## 0. Quick-Reference Traceability Table

| Addon | Exact source command(s) | File | Sourcing strength |
|---|---|---|---|
| F1 Claims Integrity Gate | `$ANOMALY_SET[claim_review]` bundle (factual+evidentiary+persuasion+meta) + `STE:WARN` | Anomaly file; `PROTOCOL_STANDARD_LIBRARY.md` Addendum v2 | Full |
| F2 Numeric Derivation Ledger | `AUDIT:MATH` | `PROTOCOL_STANDARD_LIBRARY.md`, Audit Master Prompt Library v1 | Full for the check; the standing-ledger/expiry structure is **not** from these 3 files (see §1) |
| F3 Spec & Acceptance-Criteria Ledger | `SPEC:LOCK`, `SPEC:DIFF`, `TEST:CASES`, `TEST:GAP` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v3/v4 | Full — near-literal port |
| F4 Documentation Drift Monitor | `DOC:SYNC` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v4 | Full for the check; the release-tag scheduling trigger is **not** from these 3 files |
| F5 Debt & Provenance Log | `BUILD:DEBT`, `AUDIT:CORE` (Step 0 classification) | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v3; Audit Master Prompt Library v1 | Full |
| F6 Self-Hosted Boundary Check | `$ANOMALY_SET[security]` (49–51), `AUDIT:THREAT`, `VERIFY:PROOF` | Anomaly file; `PROTOCOL_STANDARD_LIBRARY.md` v1; `MASTER_CHAIN_REFERENCE.md` Group III (one-liner only) | Mixed — see §1, weakest link flagged |
| Pre-Repackaging Dependency Map | `EXTRACT:GRAPH` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v5 | Full |
| Defect Triage | `ISSUE:TITLE/TRIAGE/BLAST` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v3 | Full |
| Code Review Gate | `PEER:REVIEW/MERGE` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v4 | Full |
| Prompt-Eng. Discipline for γ | `FORGE:PROMPT/ROLES/AUDIT` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v3 | Full |
| Schema/Format Conformance | `EXTRACT:SCHEMA`, `MORPH:FORMAT` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v5 | Full |
| Reversibility-First Decision Check | `DECIDE:GATE` | `MASTER_CHAIN_REFERENCE.md`, one-liner only | **Thin** — no full procedure available |
| Stakeholder-Impact Check | `LENS:STAKE` | `MASTER_CHAIN_REFERENCE.md`, one-liner only | **Thin** — no full procedure available |
| Pre-Mortem Risk Pass | `STRAT:PREMORTEM` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v5 | Full |
| Multi-Source Reconciliation | `SYNTH:BRIEF`, `SYNTH:CONFLICT` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v5 | Full |
| Audience-Register Discipline | `MORPH:REGISTER` | `PROTOCOL_STANDARD_LIBRARY.md` Addendum v5 | Full |
| Competitive-Monitoring Cadence | — | — | **None** — inherited from Themis Ledger's own existing practice |
| GTM strategy table (7 items) | — | — | **None** — sourced from `strategy_for_manipulation.txt`, a different file entirely |

---

## 1. Product Components (F1–F6) — Detailed Mapping

**F1 — Claims Integrity Gate**
Source: the anomaly file's own preset bundle `$ANOMALY_SET[claim_review] = factual(7–11) + evidentiary(16–20) + persuasion(26–31) + meta(45–48)` — this bundle definition is reused directly, just renamed for this context. The four category lists it pulls in: `FALSEHOOD/FALSE_CLAIM/OUTDATED_FACT/MISATTRIBUTION/PARTIAL_TRUTH` (7–11), `UNVERIFIED_VALUE/CHERRY_PICKING/SAMPLE_OF_ONE/CORRELATION_AS_CAUSATION/UNFALSIFIABLE_CLAIM` (16–20), `MANIPULATION/FALSE_CONFIDENCE/SYCOPHANCY/APPEAL_TO_AUTHORITY/PADDING_AS_RIGOR/EMOTIONAL_FRAMING` (26–31), `UNVERIFIED_SELF_CLAIM/CONFIDENCE_INFLATION/ANCHOR_BIAS/REPETITION_AS_PROOF` (45–48). Separately, `STE:WARN`'s three-step pattern (lead with a risk-level word → state the condition/command first → state the risk second, never buried after a hedge) is the direct source of the phrasing requirement on severity/limitation sentences.
**Transformation:** the bundle was originally written as a self-check a model runs on its own draft before showing it to a reader. F1 moves the identical four-category checklist from "a review step on a chat response" to "a required-clear field set in a database" — `claims_findings.status` can't reach `resolved` while any of those four categories has an open hit. No category was added or dropped; only the enforcement mechanism changed.

**F2 — Numeric Derivation Ledger**
Source: `AUDIT:MATH`'s seven-step procedure — inventory every numeric claim, re-derive independently three separate times, unit-discipline check, cross-location consistency check, real-world anchor check, downstream-use match, weakest-link trace.
**Transformation:** `AUDIT:MATH` as written is a reactive, on-demand audit — someone hands over a document, the numbers get re-derived by hand. F2 keeps the re-derivation discipline but converts it into a standing precondition: a number can't enter a document at all unless it already has a stored, dated source. **Not from these 3 files:** the `expires_at` field and the whole "standing ledger with a freshness window" structure is transplanted from a different, separate Themis Ledger component (its own whitelist-freshness mechanism), not from `AUDIT:MATH` or anything else in the protocol/anomaly corpus.

**F3 — Specification & Acceptance-Criteria Ledger**
Source, near-literal: `SPEC:LOCK`'s output contract already writes acceptance criteria as an ID-tagged list (`C1`, `C2`, ...) — this literally is the `spec_criteria` table's shape. `TEST:CASES`'s output contract already includes a column pairing each criterion ID to a case, and an explicit output row named "Criteria With Zero Cases" — this is, verbatim in structure, the `test_traceability` table and the "can't tag release-ready with zero linked tests" rule. `TEST:GAP` supplies the diff-against-behavior logic feeding back into `TEST:CASES`. `SPEC:DIFF`'s walk-each-criterion-ID-and-mark-met/not-met/partial procedure is the direct source of the release-gate check itself.
**Transformation:** minimal — these four commands were already written as ID-based, relational, traceable structures. F3 is closer to a direct port into a schema than an adaptation.

**F4 — Documentation Drift Monitor**
Source: `DOC:SYNC`'s procedure — extract every checkable claim from existing docs, verify each against current code/artifact, classify accurate/drifted/never-accurate/unverifiable, produce corrected text ready to paste in.
**Transformation:** the four-way classification becomes `doc_claims.status` directly; the "corrected text ready to paste" behavior becomes the job's output. **Not from these 3 files:** `DOC:SYNC` itself is described as an on-demand audit with no stated trigger cadence — the decision to run it specifically "on every release tag" is not in the source command; that scheduling choice is borrowed from a different Themis Ledger component's own recurring-recheck pattern.

**F5 — Technical-Debt & Provenance Log**
Source, two commands stitched together: `BUILD:DEBT`'s output contract (`shortcut — reason — cost — repay before ship? y/n`, plus its own explicit "this section is never silently empty" rule) maps directly onto `debt_log`'s columns and the enforcement logic. `AUDIT:CORE`'s Step 0 ("classify before verifying: Borrowed / Original / Bare design choice") maps directly onto `component_provenance.classification`.
**Transformation:** near-literal on both halves — both source commands already specify a structured record, not prose; F5 mainly moves that record from a chat output contract into a repo-enforced field.

**F6 — Self-Hosted Boundary Integrity Check**
Source: the anomaly file's Security Integrity category, `INJECTION_SURFACE` (49) / `PRIVILEGE_ESCALATION_BLINDNESS` (50) / `SECRET_EXPOSURE` (51) map one-to-one onto the three named checklist items (undeclared outbound calls / an over-privileged default / an embedded credential). `AUDIT:THREAT`'s procedure (enumerate attack surface, map trust boundaries, construct the actual adversarial input, trace blast radius, check detectability, supply-chain check) is the source of "a red-team pass against the declared attack surface," compressed from its full six-step form. `VERIFY:PROOF` is cited by name only — its one-line summary ("prove the result step by step") is all that was available in the supplied corpus.
**Transformation:** the security-category constants were relabeled as concrete checklist items chosen specifically because they match Themis Federal's actual self-hosted risk profile; `AUDIT:THREAT`'s full procedure was compressed rather than reproduced step-by-step. `VERIFY:PROOF` is the thinnest link in this component — flagged, not hidden.

---

## 2. Engineering Practices — Detailed Mapping

- **Pre-Repackaging Dependency Map** ← `EXTRACT:GRAPH` (identify entities, identify every relationship, flag orphans). Applied with no modification — the command was already written generally enough ("a document, codebase, or dataset") to point straight at the existing engine. The *timing* decision (run it before the port begins, not after) is not in the source command; it came from a finding in an earlier round of this project's own analysis, not from the protocol/anomaly files themselves.
- **Defect Triage** ← `ISSUE:TITLE/TRIAGE/BLAST` (title synthesis; failure/component/trigger/severity/certainty/evidence/missing-info decomposition; fix blast-radius estimate). Direct port, no modification — the source commands already assume a real report as input.
- **Code Review Gate** ← `PEER:REVIEW/MERGE` (three fixed passes — correctness/maintainability/blast-radius; "the count is the gate — it isn't re-litigated here as a judgment call"). Direct port, including that exact enforcement language.
- **Prompt-Engineering Discipline for γ** ← `FORGE:PROMPT/ROLES/AUDIT` (undefined success criteria / missing output format / self-contradiction / ambiguity checks). Direct application to γ specifically, since γ is an existing LLM-facing component this checklist was already built for.
- **Schema/Format Conformance Checking** ← `EXTRACT:SCHEMA` + `MORPH:FORMAT`, chained exactly as the source's own stated chaining rule specifies ("EXTRACT:SCHEMA >> MORPH:FORMAT — lock the shape, then convert it safely"). No modification.

---

## 3. Business Practices — Detailed Mapping

- **Reversibility-First Decision Check** ← `DECIDE:GATE`. **Thin source:** only the one-line index summary was in the supplied corpus ("Reversibility-first go/no-go"); no full procedure was available to port from. The practice as written is built from that single line's principle, not from a fuller specification.
- **Stakeholder-Impact Check** ← `LENS:STAKE`. **Thin source**, same limitation — one-liner only ("Who's affected, and how does it land on each"). The three named stakeholders (agency end-users, disability-rights stakeholders, integrator's reputation) are this project's own application of that one line, not something the source specified.
- **Pre-Mortem Risk Pass** ← `STRAT:PREMORTEM`, full procedure available and closely followed: assume total failure, work backward to root causes, categorize Technical/Organizational/Market/External, rate probability and impact honestly ("resist the pull to rate everything High to look thorough" — carried over near-verbatim), top three get a prevention and a separate mitigation, check against a named critical path.
- **Multi-Source Reconciliation** ← `SYNTH:BRIEF` + `SYNTH:CONFLICT`. The "checking whether apparent agreement is real corroboration or just two documents quoting the same origin" line is a direct restatement of `SYNTH:CONFLICT`'s own "Shared-Origin Check" step.
- **Audience-Register Discipline** ← `MORPH:REGISTER`'s R1–R5 scale. "Integrator's engineers" = R4 (domain practitioner, full vocabulary, no hand-holding) and "procurement executive" = R5 (impact/cost/risk/timeline only, implementation detail removed entirely, not just de-emphasized) — both are named levels on the source's own scale, not invented categories.
- **Competitive-Monitoring Cadence** — **no source in these 3 files.** Inherited unchanged from an existing Themis Ledger practice.

---

## 4. Items With No Protocol/Anomaly Source At All

- **GTM strategy table** (Beachhead, Bridgehead, Parasitic/Become-Their-Asset, Foot-in-the-Door, Value-Metric Pricing, Social Proof, Loss Aversion) — every one of these is sourced from `strategy_for_manipulation.txt`, a completely separate file with no connection to the protocol library or the anomaly set.
- **Competitive-Monitoring Cadence** — as above, inherited from Themis Ledger's own existing operating practice.

---

## 5. Honestly Weak Links, Stated Once More

`DECIDE:GATE`, `LENS:STAKE`, and `VERIFY:PROOF` were all cited by name and one-line summary only — their full procedures were never present in the uploaded corpus, only referenced by `MASTER_CHAIN_REFERENCE.md`'s index as existing elsewhere. Everything built on them (the Reversibility-First Decision Check, the Stakeholder-Impact Check, and part of F6) is accordingly the thinnest-sourced material in this whole mapping — real, usable, but resting on a one-line description rather than a full specification the way `SPEC:LOCK`, `DOC:SYNC`, `STRAT:PREMORTEM`, and the rest are.

## 6. Final Verdict

Fourteen of seventeen addons trace to a full, available procedure in the three named files, with the transformation logic stated for each. Three rest on one-line summaries only, flagged rather than dressed up as fully sourced. Two items in the operating model (Competitive-Monitoring Cadence, the GTM table) have no protocol/anomaly source at all and are labeled as such rather than folded in silently. **Every addon's actual provenance is now traceable to a specific command, category, or one-liner — or explicitly marked as coming from somewhere else.**
