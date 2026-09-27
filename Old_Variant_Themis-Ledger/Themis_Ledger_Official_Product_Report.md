# Themis Ledger — Product & System Report (Current Version)

**Status:** Finalized specification — architecture and strategy locked, pre-implementation.
**Working codename:** *Themis Ledger* (Themis — Greek titaness of law and evidence; not yet trademark-cleared, see §17).
**Document lineage:** `A2A_new_idea.txt` idea #11 (raw pitch) → Part 2 dual-persona audit (won the batch; pivoted to evidence layer) → v2 reconstruction (hardened against `Pandemonium`/`Osiris`/`Pandora`; sharpened against `strategy_for_manipulation.txt`) → **this document** (single current-state source of truth; supersedes nothing, consolidates everything).

---

## 1. Executive Summary

Themis Ledger is not an accessibility auto-fixer. The fixing layer (ARIA-tag patching, overlay widgets) is now commodity — free and paid MCP servers and GitHub bots already do it, and one of the category's largest players (accessiBe) was fined $1M by the FTC for overclaiming compliance through exactly that mechanism. Themis Ledger instead sells the thing none of those tools produce: an immutable, timestamped, court-ready record that a merchant ran an ongoing WCAG program. It is priced against the cost of a lawsuit ($60,000–$200,000+), not against a free tool, distributed through the platforms and consultancies that already hold the merchant relationship, and built so that its highest-stakes action — writing to a merchant's live storefront — is gated by deterministic rules, not model judgment, at every step.

## 2. Design Principles

1. **The evidence is the product, not the fix.** Every mechanism ultimately serves the Evidence Ledger (β); remediation is a secondary, tightly-gated feature layered on top of it.
2. **Never claim what can't be backed up.** No "compliant" label, no percentage score, no certification badge — permanently, not just at launch. This is the direct, structural fix for the failure mode that got a market leader fined.
3. **Every automated write has a deterministic ceiling.** A whitelist decides what's allowed to auto-commit, never a model's confidence; a ring classification makes an entire category of code (checkout/payment flow) structurally unreachable, with no exception path.
4. **Complexity earns its way in with proof, not by default.** New capability enters production only through a named, evidence-based promotion path — never by developer fiat, never by merchant pressure alone.
5. **The channel is trust, not ad spend.** Distribution runs through platforms and partners that already hold the merchant relationship, not through outbidding funded incumbents on marketing spend.

## 3. Market Context

Federal ADA/WCAG-related digital-accessibility lawsuits reached 3,117 filings in 2025, up 27% year over year, pacing toward roughly 6,176 in 2026 — 79% of which hit e-commerce specifically. The category's highest-profile compliance vendor, accessiBe, was fined $1,000,000 by the FTC for advertising its overlay widget as delivering full compliance it didn't. That fine is the market's clearest signal of the exact positioning trap Themis Ledger is built to avoid.

## 4. Product Positioning

| What competitors sell | What Themis Ledger sells |
|---|---|
| Auto-fixed ARIA tags / overlay widgets | Timestamped, exportable evidence of an ongoing accessibility program |
| A compliance score, badge, or "certified" claim | No compliance claim of any kind — evidence of diligence only |
| A one-time manual audit ($2,500–$10,000) | Continuous monitoring ($99–$299/mo), sold *through* the same consultancies that used to sell the one-time audit |
| Competing for visibility among many similar apps in a marketplace | Pursuing official platform-partner status — becoming the recommended layer, not one of many |

## 5. Target Customer & Beachhead

Shopify merchants first — the single platform with both the largest share of 2026 filings (79% of all filings are e-commerce) and the deepest existing app-store distribution surface. WooCommerce is the deliberate second platform, not a launch requirement. No other vertical or platform is in scope for the current version.

## 6. System Architecture — Runtime Flow

```
 merchant storefront ──▶ ι Platform Adapter (Shopify webhook; WooCommerce v2)
                              │
                              ▼
                    α Scan Orchestrator (axe-core, scheduled + on-publish)
                              │
                              ▼
                    β Evidence Ledger (immutable, timestamped — the actual asset)
                              │
        ┌─────────────────────┼───────────────────────┐
        ▼                     ▼                        ▼
  ν Ring Classifier      ε Regression Watchdog     ζ Digest & Notification
  (0 / 1 / 1.5 / 3)      (re-checks old fixes)     (plain counts, never a %)
        │
   ┌────┼─────────────────────┐
   ▼    ▼                     ▼
 Ring 0  Ring 1              Ring 1.5
 (log    γ Fix-Suggestion    κ Deterministic Auto-Fix Gate
 only,   Engine ──▶ ξ         gated on ALL of:
 no      Verification          ├─ ο Cumulative-Risk Ledger (per-run cap)
 action) (2-path agreement)    ├─ ρ Whitelist Freshness + Canary Suite
             │                 ├─ π Disjoint Backup Pre-Compute (rollback staged first)
             ▼                 ├─ τ Ephemeral Scoped Write-Token
       merchant review         └─ φ Idempotency Key
       (υ batches if                │
        volume is high)             ▼
             │                commit to storefront
             ▼
       σ Outcome-Calibrated Whitelist Expansion
       (accept/reject labels are the ONLY sanctioned
        path to widen κ's whitelist)

 Ring 3 — checkout / payment-flow code: no path into this diagram exists. Refused by design.

 ── Distribution & content layer (not part of the runtime pipeline) ──
   θ Agency Partner Program        χ Platform-Asset (Shopify partner) track
   δ Accessibility Statement       η Demand-Letter Exporter    λ VPAT/ACR (on demand, from β)
   μ Aggregate Benchmark Report (periodic, built entirely from anonymized β data)
   ω Foot-in-the-Door funnel: free scan → paid monitoring → agency/VPAT tier
   ψ Honest named-case trust content (real, consented case studies only)
```

## 7. Ring Classification (ν)

| Ring | Scope | Gate | Example in this product |
|:---:|---|---|---|
| 0 | Read-only, zero consequence | None — logs directly to β | Running a scan |
| 1 | Suggests a change, commits nothing | ξ verification (2-path agreement) before it reaches a human | γ's fix suggestions |
| 1.5 | Writes to the merchant's live storefront | κ's full gate stack: ο + ρ + π + τ + φ, every time, no exceptions | Auto-committing a whitelisted alt-text fix |
| 3 | Checkout / payment-flow code | **Refused by design** — no automated or suggested path touches this code at all | N/A — permanently out of scope |

## 8. Full Component Catalog

| Symbol | Component | Mechanism | Ships in |
|:---:|---|---|---|
| α | Scan Orchestrator | Runs `axe-core` on a schedule and on every theme-publish webhook | v0 (MVP) |
| β | Evidence Ledger | Immutable, timestamped store of every raw scan result | v0 (MVP) |
| γ | Fix-Suggestion Engine | LLM-guided suggestions for anything outside κ's whitelist; never auto-committed | Month 1 |
| δ | Statement & Feedback-Form Generator | Templated accessibility statement + reporting-path page | v0 (MVP) |
| ε | Regression Watchdog | Re-checks that a previously-fixed issue hasn't silently returned | Month 1 |
| ζ | Digest & Notification Engine | Plain-count monthly digest — facts only, never a percentage or "certified" claim | v0 (MVP) |
| η | Demand-Letter Response Exporter | One-click evidence bundle export the moment it's actually needed | Month 1 |
| θ | Agency Partner Program | White-label resale to existing manual-audit consultancies | Month 1 |
| ι | Platform Adapter Layer | Shopify webhook (v1), WooCommerce plugin (v2) | v0 (MVP) / Year 1 |
| κ | Deterministic Auto-Fix Gate | Hard whitelist — only provably-safe fixes ever auto-commit | Month 1 |
| λ | VPAT/ACR Report Generator | Formal conformance report for enterprise/agency buyers | Year 1 |
| μ | Aggregate Benchmark Report | Reuses β's data for an annual lead-gen content asset | Year 1 |
| ν | Ring Classification (0/1/1.5/3) | Every action assigned a ring; checkout/payment code structurally unreachable | v0 (MVP) |
| ξ | Verification Pass | Two independent generations must agree before a γ suggestion surfaces | Month 1 |
| ο | Cumulative-Risk Ledger | Per-run cap on total auto-committed changes; crossing it forces human review | Month 1 |
| π | Disjoint Backup Pre-Compute | Verified rollback staged and confirmed *before* any κ commit fires | v0 (MVP) |
| ρ | Whitelist Freshness + Canary Suite | Every κ rule carries a last-verified date; stale rules auto-downgrade to suggestion-only | Month 1 |
| σ | Outcome-Calibrated Whitelist Expansion | Real merchant accept/reject data is the only sanctioned path to widen κ | Year 1 |
| τ | Ephemeral Scoped Write-Token | κ's one write action runs on a short-lived, single-purpose token, never a standing key | Month 1 |
| υ | Batched Review Queue | Clusters similar pending reviews across an agency partner's stores instead of N separate asks | Year 1 |
| φ | Idempotency Key | A retried write with the same key is a no-op if already applied | v0 (MVP) |
| χ | Platform-Asset GTM | Pursuing Shopify's official-partner track, not just app-store listing | Year 1 |
| ψ | Honest Named-Case Trust Content | Real, permission-granted case studies (or clearly-labeled composites) in marketing | Year 1 |
| ω | Foot-in-the-Door Funnel Sequencing | Deliberately small ask-size ladder: free scan → paid monitoring → agency/VPAT tier | v0 (MVP) design principle; matures through Year 1 |

## 9. Technical Stack & Explicit Non-Requirements

**Core stack:** `axe-core` (open-source WCAG scan engine), a standard relational database (Postgres), and REST/webhook integration with the Shopify Admin API (WooCommerce REST API in v2). Nothing exotic.

**Explicitly not needed, and why:**
- **No graph database or graph-search algorithm.** Every relationship in the data model is one-to-many (store → scans → issues → fixes), fully expressible with foreign keys. Nothing in the pipeline performs traversal, path-finding, or dependency-graph reasoning.
- **No HNSW or vector index.** There is no semantic similarity search or embedding-based retrieval anywhere in this product. γ generates a suggestion for one specific, already-identified violation — it never retrieves one via nearest-neighbor lookup against a corpus.
- **No Bloom filter.** κ's whitelist and ρ's freshness table are small (dozens of rows), exact-match, indexed lookups. A Bloom filter solves probabilistic membership testing against a large, memory-constrained set — a problem this product's scale doesn't have.

## 10. Data Model (sketch)

| Table | Key fields | Feeds |
|---|---|---|
| `stores` | store_id, platform, domain, plan_tier, agency_partner_id (nullable) | ι, θ |
| `scans` | scan_id, store_id, run_at, trigger_type, axe_core_version, wcag_ruleset_version | α, β |
| `issues` | issue_id, scan_id, rule_id, wcag_criterion, severity, element_selector, status | β, ν |
| `fixes` | fix_id, issue_id, fix_type, applied_at, rollback_snapshot_id, idempotency_key, approved_by | κ, π, φ |
| `rollback_snapshots` | snapshot_id, store_id, theme_file_ref, captured_at | π |
| `whitelist_rules` | rule_id, rule_type, last_verified_at, status | κ, ρ |
| `outcome_labels` | issue_id, suggestion_id, merchant_decision, labeled_at | σ |
| `agency_partners` | partner_id, name, managed_store_ids, tier | θ, υ |
| `benchmark_aggregates` | period, anonymized_stats_json | μ |

## 11. MVP Scope & Phased Roadmap

- **v0 / MVP (week 1):** α, β, δ, ζ ship as the free lead-magnet scan — no login, no compliance claim. ν, π, φ ship alongside it as invisible internal hardening, since they're cheaper to build in from day one than retrofit later.
- **Month 1:** κ's whitelist opens (alt-text-from-existing-data only), backed by ξ, ο, ρ, and τ from first commit. γ, ε, η go live. First 2–3 Agency Partners (θ) onboarded at a discount for their client base and a case-study reference.
- **Year 1:** λ ships for the enterprise/agency tier. ι v2 (WooCommerce) goes live. σ begins reclassifying suggestion types into κ based on real accept/reject data. υ activates once partner volume justifies batching. χ (platform-partner pursuit) and ψ (honest case-study content) mature alongside μ's first annual benchmark report.

## 12. Revenue Model

$99–$299/month per store, tiered by SKU/page count, sold through the Shopify and WooCommerce app stores (standard platform revenue share accepted as the cost of that distribution channel). θ adds a white-label tier priced per managed store for agency partners, replacing their one-time $2,500–$10,000 manual audit with a recurring retainer product.

## 13. Go-to-Market Strategy

Beachhead on Shopify → free scan as Trojan-horse/engineering-as-marketing lead magnet → deliberately staged foot-in-the-door funnel (ω: free scan → paid monitoring → agency/VPAT tier) → content moat via μ's annual benchmark report → Agency Partner Program (θ) turning existing manual-audit consultancies into resellers instead of competitors → a platform-level asset play (χ) pursuing Shopify's official-partner track rather than competing for app-store search visibility alone → honest, consented case-study content (ψ) → pricing-page mechanics anchored against real settlement costs, a 3-tier structure with a clear rational middle option, and annual billing as the default. Posture throughout is Fabian, not Blitzkrieg: no attempt to outspend funded overlay-widget incumbents on advertising — win on app-store organic ranking and inbound content instead. One additional standing practice, not a lettered component: an ongoing, weekly-cadence competitive-monitoring pass on the accessibility-tooling market, run with the same discipline as the separate RFP-engine project's competitor-scanning pattern, so a shift in the category gets caught in weeks, not quarters.

## 14. Full Checkpoint Compliance (current state)

| # | Checkpoint | Status | Note |
|---|---|:---:|---|
| 1 | Practical | ✅ | Lawsuit exposure is a real, board-level line item |
| 2 | 100% buildable | ✅ | `axe-core`, Postgres, standard webhooks — nothing unproven |
| 3 | Grounded, no sci-fi | ✅ | — |
| 4 | Unavoidable for the crowd | ⚠ | Scoped to e-commerce merchants — the highest-leverage slice, not literally everyone |
| 5 | Thoroughly checked | ✅ | Verified lawsuit data, a named FTC case, a direct competitive scan, and a cross-check against two independent architecture/strategy corpora with explicit rejections recorded |
| 6 | Hard to refuse | ✅ | Priced against a $60K–$200K settlement, not against a free tool |
| 7 | Easy to onboard | ✅ | App-store install, zero merchant configuration for the MVP |
| 8 | Efficient | ✅ | Reuses `axe-core` rather than building a scanner from scratch |
| 9 | Maps to unavoidable cost/pain | ✅ | Legal settlement and defense cost is the pain being displaced |
| 10 | No new build where one exists | ✅ | α reuses proven OSS tooling; ν–φ are borrowed patterns, not reinvented machinery |
| 11 | Stays honest about its own limits | ✅ | See §16 and §17 |
| 12 | Beatable competition / overlooked issue | ✅ | Not "no competition" — competitors solve the fixing layer; this solves the evidence layer |
| 13 | Strongest defensible solution | ✅ | Two full pivots already executed (auto-fixer → evidence layer → hardened + platform-asset GTM) |
| 14 | Features are MVPs | ✅ | v0 is deliberately four components, no more |
| 15 | Balance complexity | ✅ | ν–φ are invisible hardening, not merchant-facing features — see §16 |
| 16 | Near-zero-complexity trust building | ✅ | Free scan, no login, no compliance claim — the accessiBe trap avoided by design |
| 17 | Keep hitting the field's pain points | ✅ | δ/ε/ζ/η/λ each target a distinct, named lawsuit-relevant failure mode |
| 18 | Too helpful to avoid, not engineered dependency | ✅ | Every dependency-engineering GTM tactic considered was explicitly named and rejected |
| 19 | Seep product root + tactical GTM | ✅ | θ + ι + χ together, see §13 |
| 20 | 1wk/1mo/1yr ripple diagnostics | ✅ | See §11 |
| 21 | Deterministic safety gates | ✅✅ | ν's ring ceiling, κ's whitelist, π's pre-staged rollback, ο's cumulative cap, τ's scoped token, φ's idempotency check — no single point of judgment-based failure |
| 22 | Local redaction before cloud | N/A-ish | Scanned content is public storefront markup, not personal/financial/medical data; the one live caveat is not to capture customer PII if a scan ever touches a logged-in checkout state |
| 23 | Enter via highest-trust channel | ✅ | The app store itself is the trust channel; χ extends this to platform-partner status |
| 24 | Radical simplicity | ✅ | Zero-config install, plain-count digest, no dashboard required for v0 |
| 25 | Near-zero new data-entry | ✅ | μ and κ both reuse data the pipeline or merchant already generated |
| 26 | Deterministic query first | ✅ | γ/ξ suggest, κ deterministically decides what's allowed to commit |
| 27 | Parasite: become the incumbent's asset | ✅ | θ turns manual-audit consultancies into resellers; positioned as a complement to free MCP fixing tools, not a replacement |
| 28 | Bind competitor in "optimal solution" dilemma | ✅ | A consultancy either partners via θ or watches its own one-time audit look obsolete next to continuous monitoring |

## 15. Honesty Scores (current)

| Dimension | Score |
|---|:---:|
| Reality | 8 |
| Uniqueness | 7 |
| Paper-worthiness | 7 |

## 16. Explicitly Out of Scope

- Any write path into checkout or payment-flow code (Ring 3) — permanent, not a v1 limitation.
- Graph databases, graph-search algorithms, HNSW/vector indexing, and Bloom filters — none of the product's actual problems call for them (§9).
- Auto-fix beyond κ's calibrated whitelist — expansion happens only through σ's evidence-based process, never by developer fiat or merchant pressure.
- Deceptive or dependency-engineering GTM tactics considered and rejected during the reconstruction pass (astroturfing, dark PR, fake urgency/cancellation friction, and related patterns) — several are outright illegal, and all of them fail checkpoint 18 on its own terms.
- Any hardware-enclave, GPU-kernel, or activation-level jailbreak-detection machinery from the adjacent `Pandemonium`/`Osiris`/`Pandora` corpus — solves a threat model this product doesn't have.
- Any platform beyond Shopify (v1) and WooCommerce (v2) for the current version.

## 17. Open Items — Not Yet Decided

- Exact per-tier pricing needs real willingness-to-pay validation from actual discovery calls, not just the anchoring logic in §13.
- No named Agency Partners (θ) have been identified or contacted yet.
- Marketing copy using language like "evidence of diligence" needs an actual legal review before any public launch — this is a hard prerequisite, not a nice-to-have, given the category's own accessiBe precedent.
- The WCAG 2.2→3.0 transition timeline is something ρ needs to monitor once live; its practical impact on the whitelist isn't fully known yet.
- "Themis Ledger" is a working codename only — not brand- or trademark-cleared.

## 18. Version Lineage Reference

| Stage | Source | What it was |
|---|---|---|
| Raw pitch | `A2A_new_idea.txt` idea #11 | An LLM code-fixer API / CI-CD compliance gate — both already crowded at the fixing layer |
| Pivot 1 — evidence layer | Part 2 dual-persona audit | Reframed as system-of-record; won the batch-2 idea audit against ~19 other candidates |
| Pivot 2 — hardening + GTM | v2 reconstruction | ν–ω added from `Pandemonium`/`Osiris`/`Pandora` patterns and from `strategy_for_manipulation.txt`'s legitimate GTM plays, with explicit rejections recorded for both corpora |
| **Current** | This document | Single consolidated source of truth for the finalized architecture and strategy |
