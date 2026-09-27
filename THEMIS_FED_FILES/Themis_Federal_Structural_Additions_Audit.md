# Themis Federal — Structural Additions Audit [depth=10]

**Scope of this file:** only mechanisms from the protocol library / anomaly set that become an actual part of Themis Federal — a software component, a data table, a CI gate, or a standard engineering practice on the codebase. Nothing about drafting style, session behavior, or how any assistant talks to anyone. If it doesn't show up in the product or the repo, it isn't in this file.

---

## 1. New Product Components (F1–F6)

Named F1–F6 (Themis Ledger's own α–ω is already the full Greek alphabet — no collision risk this way). Each entry: mechanism, data model, where it sits in the pipeline, source, honest limitation.

| # | Component | Mechanism | Data model | Integration point | Source | Limitation |
|---|---|---|---|---|---|---|
| F1 | **Claims Integrity Gate** | Scans every δ-statement, λ-VPAT/ACR, and RFI-facing document before "final" status: unsourced numeric claims, certainty language exceeding actual evidence strength, coverage claims that omit axe-core's ~30–50% WCAG-detection ceiling, "we verified X" with no attached verification record. Severity/limitation sentences must be risk-first phrased (condition before consequence). | `claims_findings(doc_id, claim_text, anomaly_type, status)` — blocks `published` while any row is `unresolved` | Between λ's report generation and export/delivery — same choke-point pattern as κ's gate stack | `$ANOMALY_SET` (factual, evidentiary, persuasion, meta categories) + risk-first phrasing pattern | Pattern-based, not semantic — narrows, doesn't eliminate, the miss rate on a cleverly-worded overclaim; human sign-off still required |
| F2 | **Numeric Derivation Ledger** | Every statistic in an RFI response or VPAT (a lawsuit-count figure, an axe-core coverage %, an integrator's claimed capability) must have a stored source row with citation + date-checked. F1 blocks publish on a mismatch. | `verified_numerics(claim_id, value, source_url, checked_at, expires_at)` | CI/build-time check, not a runtime call | Numerical re-derivation discipline | Only as current as the last recheck — `expires_at` forces re-verification rather than trusting a stale figure indefinitely, same pattern as ρ's whitelist-freshness applied to citations |
| F3 | **Specification & Acceptance-Criteria Ledger** | Every packaging/build phase (auth model, telemetry-off default, licensing gate) gets ID-tagged criteria before work starts; each criterion maps to at least one automated test; a build can't tag `release-ready` if any criterion has zero linked tests. | `spec_criteria(phase_id, criterion_id, text, status)`, `test_traceability(criterion_id, test_id)` | CI pipeline gate | Criterion-ID spine + criterion-traceable test generation + spec-vs-delivered diff | Only as good as how testably the criteria were written in the first place |
| F4 | **Documentation Drift Monitor** | Scheduled job re-checking every claim in shipped docs (install guides, integrator docs, the VPAT's own coverage-scope disclosure) against current code behavior, on every release tag. | `doc_claims(doc_id, claim_text, verified_against_version, status)` | Release-tag CI job | Documentation-vs-behavior drift audit | Needs docs decomposed into checkable claims up front — a monolithic prose doc can't be diffed this way without that structuring |
| F5 | **Technical-Debt & Provenance Log** | Every shortcut in the self-hosted repackaging is a required, structured field on every merged change — never optional. Every component is classified Borrowed (real cited source) / Original / Bare-design-choice, the same discipline Themis Ledger's own stack docs already apply informally to axe-core, Postgres, etc., now a checked field instead of prose. | `debt_log(change_id, shortcut, cost, repay_before_ship: bool)`, `component_provenance(component_id, classification, source_ref)` | Repo/CI tooling; a merge with `repay_before_ship=true` unresolved blocks a release tag | Debt-disclosure discipline + provenance classification | Depends on the person merging the change filling it in honestly — a discipline, not a technical guarantee |
| F6 | **Self-Hosted Boundary Integrity Check** | Deployment lands inside the integrator's own accredited environment, so a fixed pre-handoff checklist runs against the package: no outbound network calls beyond declared, no default telemetry, no embedded secrets, a red-team pass against the declared attack surface, and a step-by-step proof for any claim like "captures no telemetry by default." | `deploy_boundary_checks(build_id, check_name, result, checked_at)` | Pre-handoff release gate — parallel to κ's gate stack, applied to the shipped package instead of a live write action | Security-integrity checks + threat/blast-radius audit method | Catches known/declared-surface violations; doesn't replace the integrator's own independent security review (already a required step in the roadmap) |

---

## 2. Engineering Practices on the Codebase (currently dormant unless noted)

| Practice | What it does | Trigger | Source |
|---|---|---|---|
| **Pre-Repackaging Dependency Map** | Maps every relationship in the existing engine before the self-hosted port touches it — catches a Shopify-specific coupling before it becomes a build-time surprise. | Active now, before packaging begins | Entity/relationship mapping |
| **Defect Triage** | Standard severity/blast-radius bug triage on real reports. | First real bug report | Defect decomposition + fix-blast-radius estimate |
| **Code Review Gate** | Diff-scoped, time-boxed review with a hard must-fix gate before merge. | First commit from anyone but the solo builder | Diff-scoped review discipline |
| **Prompt-Engineering Discipline for γ** | Applies when γ's fix-suggestion prompts (reused from Themis Ledger) get revised, or a new LLM-facing feature is built specifically for Themis Federal. | First such revision/build | Prompt construction + pre-ship prompt audit |
| **Schema/Format Conformance Checking** | Validates Themis Federal's data model against a specific integrator's existing systems before handoff. | First real integrator data-model exchange | Schema inference/conformance + format conversion |

---

## 3. Critique / Resolution — Why Each Item Cleared

| Critique | Resolution |
|---|---|
| A pattern-based claims linter (F1) will miss a cleverly-worded overclaim — false confidence in the gate itself would be worse than no gate. | Stated as an explicit, permanent limitation, not a caveat that quietly disappears — human sign-off stays required regardless of F1's pass/fail. |
| A numeric ledger (F2) with no expiry just launders a stale number forever. | `expires_at` is mandatory on every row; a lapsed number re-enters "unverified" status automatically. |
| Acceptance criteria (F3) are only as good as how they're written — garbage-in, garbage-out. | `test_traceability` makes "zero linked tests" a visible, blocking state rather than a silent gap. |
| Debt disclosure (F5) depends on a person being honest when merging their own change. | No technical fix exists for this — it's named as a discipline, not a guarantee, consistent with how the rest of this project treats every other honesty-dependent gate. |
| F6's boundary check could be mistaken for a full security clearance. | Explicitly scoped as a pre-check, not a substitute for the integrator's own independent review. |

---

## 4. Excluded — Not Part of the Product, Not Carried Forward

The following had no form beyond affecting how a chat assistant drafts or reasons in a conversation, not anything inside Themis Federal itself, and are dropped entirely rather than forced into a false product mapping: session-recall conventions, response-verbosity dials, no-clarification/best-effort drafting behavior, requirement-scoping shorthand used only for framing a conversation, multi-advisor decision-drafting techniques, general prose-ambiguity rewriting rules, and any command whose only demonstrated use across this project was shaping a written reply rather than the software or the business.

Separately, and unrelated to the above: the source `PROTOCOL_STANDARD_LIBRARY.md` file contains an embedded instruction-override block. It is not integrated in any form, was never executed, and is excluded permanently. Strategy-catalog items involving disinformation, astroturfing, personnel-planting, or fake-authority coercion are excluded on the same permanent basis — illegal or dishonest, not situationally inapplicable.

---

## 5. Checkpoint Coverage

F1/F2 → checkpoints 6, 11, 16 (never claim what can't be backed up). F3/F4 → checkpoints 2, 5, 14 (buildable, thoroughly checked, testable done-definition). F5 → checkpoints 10, 11 (reuse over reinvention, honest about limits). F6 → checkpoint 21 (deterministic safety gates) applied to a shipped package instead of a live write action. Engineering practices (§2) → checkpoint 8 (efficient — dormant costs nothing until triggered).

## 6. Final Verdict

Six real components, five real dormant-or-active engineering practices, each with a mechanism, a data model or repo artifact, and a named limitation — not a style guide. Everything without a product or repo form was cut rather than relabeled to look like one. **Cleared as the actual structural-addition layer on top of the reused Themis Ledger core.**
