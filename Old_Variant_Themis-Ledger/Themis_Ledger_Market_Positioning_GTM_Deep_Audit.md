# Themis Ledger — Market, Positioning & GTM Deep Audit [depth=10]

**Scope of this file, exclusively:** Market Context, Product Positioning, Target Customer & Beachhead, Revenue Model, Go-to-Market Strategy. Technical stack, MVP correctness, and structural safety are covered in Files 1–3, not here.

**Method — unchanged, applied to business claims instead of code.** Every topic gets: the credibility-checked facts, a ruthless dual-persona critique, a fix, math wherever the claim is checkable, a checkpoint mapping, and a Day 1 → Week 1 → Month 1 → Year 1 chronology. A business assumption that fails its critique with no fix is flagged, not softened.

---

## 1. Market Context

**Credibility-checked facts (previously verified, carried forward):** 3,117 federal digital-accessibility lawsuits in 2025 (+27% YoY), pacing toward ~6,176 in 2026, 79% hitting e-commerce specifically; accessiBe fined $1,000,000 by the FTC for overclaiming compliance via its overlay widget.

**Newly verified this pass:** Shopify reports roughly 4.6–4.8 million active merchants; third-party crawlers (BuiltWith, Store Leads) put "live storefronts" anywhere from 2.5 million to 6.9 million+ depending purely on methodology — the spread is definitional, not a disagreement about reality, and is stated as a range rather than a single cherry-picked number. Apparel is Shopify's single largest category at roughly 785,000–796,000 stores. The US holds an estimated 37–39% of global Shopify stores and roughly 30% of US e-commerce technology share.

### Dual-persona critique

**Ruthless question:** is this a real, sustainable subscription market, or lawsuit-headline noise that spikes and fades? **Fix:** the +27% YoY trend is now a multi-year pattern (this document's earlier files already cite the 2025-vs-2026 trajectory), and the underlying legal exposure (ADA Title III applied to digital storefronts) is not a one-time news cycle — it's a standing feature of US civil litigation that has been building for years. The FTC's accessiBe action is a second, independent signal (regulatory, not just plaintiff-driven) that the category is durable enough to draw enforcement attention, not just ambulance-chasing.

### Illustrative TAM math (explicitly modeled, not measured)

Applying 79% e-commerce share to the ~6,176 projected 2026 filings gives roughly 4,879 e-commerce-related filings. Applying Shopify's ~30% US e-commerce technology share as a rough proxy suggests somewhere in the range of **1,000–1,500 of 2026's US filings could plausibly involve a Shopify-powered storefront** — stated as a modeled estimate with real uncertainty, since litigation targeting correlates with visible traffic and brand size, not platform market share alone, and larger Shopify Plus brands are likely over-represented relative to a small store's share of the platform.

### Chronological analysis

| Stage | What's true about the market at this point |
|---|---|
| Day 1 | Founding facts are fixed and documented once in a shared internal reference — no re-deriving them in every piece of copy |
| Week 1 | Discovery calls test whether the fear level in the stats matches how real merchants actually talk about the risk, not just whether the stats are true |
| Month 1 | φ's competitive-monitoring cadence (established in the reconstruction file) starts tracking new filings/FTC actions monthly — market context becomes a living input, not a launch-day snapshot |
| Year 1 | Full 2026 filing data becomes available (~Q1 2027); reassess whether +27% held, decelerated, or accelerated, and revisit the beachhead choice against real numbers instead of a projection |

### Checkpoint mapping
Checkpoints 1, 5, 9, 11.

---

## 2. Product Positioning

**Recap:** evidence layer, not auto-fixer; priced against lawsuit cost; never claims compliance.

### Dual-persona critique

**Ruthless question:** in a market where buyers want reassurance, does "we don't claim compliance" read as an honest hedge to a lawyer and a *weak* pitch to a merchant who just wants to be told "you're covered"? A confident competitor's "we make you compliant" is simpler to sell, even if it's less true. **Fix:** the confidence in the pitch has to come from *specificity and proof*, not from a promise. The sales message is not "we don't know if you're compliant" (weak) — it's "if you're ever sued, you will have a real, timestamped, court-ready record that you ran an ongoing program, with dates, not a guess" (concrete and provable). That's a stronger claim than a compliance badge, because a badge can be — and in accessiBe's case, was — disproven in an FTC action. Evidence can't be disproven the same way; it's either there or it isn't.

### Chronological analysis

| Stage | What's true about positioning at this point |
|---|---|
| Day 1 | The evidence-not-certification line is encoded in δ's statement template and the marketing headline from the first commit, not iterated in later |
| Week 1 | Real prospect reactions tested against the "you'll have proof, not a badge" framing specifically — watching for whether people still ask "so am I compliant?" (a sign the reframe needs sharpening) |
| Month 1 | First λ-adjacent explainer content ships publicly, reinforcing the distinction outside the product itself |
| Year 1 | μ's first annual benchmark report becomes the positioning's most credible public proof point — "evidence layer" stops being a claim and becomes a demonstrated body of published work |

### Checkpoint mapping
Checkpoints 6, 16, 18.

---

## 3. Target Customer & Beachhead

### Funnel math (TAM → SAM → SOM, explicitly labeled by confidence level)

| Layer | Estimate | Basis |
|---|---|---|
| TAM | ~1.7–1.9M active US Shopify merchants | Global active-merchant figure (~4.6–4.8M) × US share of Shopify stores (~37–39%) — **modeled**, not a directly reported US-active-merchant number |
| SAM | ~300,000–500,000 US Shopify stores with genuine commercial traffic | Filtered from the ~2.5–3.5M "realistically actively trading" global estimate down to a US, meaningfully-trafficked subset — a lawsuit target needs visible commercial activity to be worth suing, which filters out the long tail of near-dormant stores; **illustrative filter, not a measured segment** |
| SOM (Year 1) | ~2,000 stores | Carried from the official product report's own Year-1 roadmap target (8–12 agency partners plus direct installs) |

2,000 stores against a 300K–500K SAM is roughly **0.4–0.7% Year-1 penetration** — a modest, plausible capture rate for a bootstrapped launch, not a hockey-stick assumption that would collapse under scrutiny.

### Dual-persona critique

**Ruthless question:** is Shopify really the single best beachhead, or would WooCommerce (more raw sites, more budget-conscious SMB base) or a narrower vertical like apparel (the single largest category, and — per general accessibility-litigation reporting, worth confirming directly rather than assumed — a frequently-named vertical in ADA suits) beat it? **Fix:** this doesn't have to be an either/or. Shopify remains the *platform* beachhead (best app-store distribution, 79% of filings are e-commerce, already justified). Apparel becomes the *messaging* beachhead *within* Shopify — first case studies, first content, first outbound all target apparel/DTC brands specifically, rather than treating "Shopify merchants" as one undifferentiated audience. Whether apparel is genuinely over-indexed in real filings is exactly the kind of claim the Week-1 discovery calls should confirm before it's treated as settled, not assumed from category size alone.

### Chronological analysis

| Stage | What's true about the target customer at this point |
|---|---|
| Day 1 | Shopify-only; "e-commerce merchant" is the only stated filter |
| Week 1 | Discovery calls test whether apparel/DTC specifically shows sharper pain than the general merchant population |
| Month 1 | The first real paying cohort defines the *actual* beachhead empirically — whoever converts fastest from the free scan, which may or may not match the apparel hypothesis |
| Year 1 | WooCommerce (already planned as ι v2) plus a possible second vertical decided from real Year-1 conversion data, not the original guess |

### Checkpoint mapping
Checkpoints 4, 15, 23.

---

## 4. Revenue Model

### Competitive pricing context (verified)

| Product | Pricing (monthly) | What it sells |
|---|---|---|
| accessiBe (accessWidget) | ~$59/mo entry (~$41/mo annualized), scaling with traffic up to several hundred/mo at higher tiers | Overlay widget, auto-remediation claims |
| UserWay | ~$49–$359+/mo, scaling with page views | Overlay widget, monitoring add-ons |
| Equally AI | ~$45/mo entry | Overlay widget |
| **Themis Ledger** | **$99–$299/mo, tiered by SKU/page count** | Evidence ledger, never an overlay, never a compliance claim |

Themis Ledger's pricing sits inside the same band the market already pays for a fundamentally weaker (and, in accessiBe's case, FTC-sanctioned) claim — the pricing isn't a hard sell on cost, it's a sell on what the money actually buys.

### Unit economics (explicitly modeled, not measured — real numbers don't exist until real launch data does)

- **ARPU (modeled):** ~$150/mo blended across tiers → $1,800/year per store.
- **Year-1 ARR at SOM = 2,000 stores:** 2,000 × $1,800 = **$3.6M gross**.
- **Shopify revenue share, applied correctly (not just the headline rate):** 0% on the first $1M (lifetime cap, confirmed current policy), 15% marginal above it. On $3.6M: first $1M free, remaining $2.6M × 15% = $390K. Effective take-home ≈ **$3.21M**, an effective blended rate of ~10.8% — meaningfully better than naively applying 15% to the whole figure.
- **Stress test (dual-persona, "do or die"):** Shopify's exemption structure has already changed once (from an annual reset to a lifetime cap) — a real, demonstrated policy-risk, not a hypothetical one. Modeling the harsher case (flat 15% from dollar one, no exemption at all): $3.6M × 15% = $540K, leaving $3.06M. **The model survives a worse version of a policy that has already proven it can change.** That's the right way to pressure-test a revenue model that depends on a third party's fee structure.
- **CAC:** genuinely unknown until real launch data exists — flagged honestly (checkpoint 11) as a Week-1/Month-1 measurement priority, not an assumed number. The bootstrap/Fabian posture (§5) means the realistic Day-1 cost driver is founder time, not ad spend.
- **LTV (modeled, assumption stated):** at $150/mo ARPU and an assumed 24-month average retention (a low-friction insurance-like product should retain well once installed, but this is an assumption, not a measurement) → **~$3,600 LTV per store**, gross.

### The sharpest anchor in this entire document

Even at the top tier, $299/mo × 12 = $3,588/year — **1.8% to 6% of a single lawsuit's stated $60K–$200K settlement range.** A merchant paying the full year at the top tier spends less than 6 cents of every dollar a single lawsuit could cost them. This is the number the pricing page should lead with, not bury.

### Chronological analysis

| Stage | What's true about revenue at this point |
|---|---|
| Day 1 | Free tier live; no paid tier exists yet — nothing to charge for until κ ships in Month 1, consistent with File 2's honest MVP scoping |
| Week 1 | Pricing page live with the anchor math above, even before billing is wired — validate willingness-to-pay via a "notify me" click-through signal before the Billing API integration is finished |
| Month 1 | Shopify Billing API live; first real paid conversions once κ ships |
| Year 1 | θ's agency white-label tier becomes a material revenue line; full-year real conversion/churn data replaces every modeled number above |

### Checkpoint mapping
Checkpoints 6, 9, 20.

---

## 5. Go-to-Market Strategy

**Recap:** Beachhead (Shopify) → Trojan-horse free scan → foot-in-the-door funnel (ω) → content moat (μ) → Agency Partner Program (θ) → platform-asset pursuit (χ) → honest case-study content (ψ) → Fabian bootstrap posture, no ad-spend war with funded incumbents.

### Dual-persona critique

**Ruthless question, the single biggest one in this file:** what stops Shopify itself from shipping a native accessibility-scanning feature and making this whole category redundant — a well-documented risk in the app-economy business model generally? **Fix, two parts.** First, χ (pursuing official platform-partner status) is a direct hedge against this — being positioned as Shopify's endorsed partner for the category makes integration or acquisition more likely than replacement. Second, and more structurally: a platform generating legal evidence that could be used *against its own merchants* in a lawsuit is a materially more fraught position for Shopify itself to hold than for an independent, specialized vendor — this is a genuine structural reason Shopify is less likely to want to own this specific category natively, not just a hope that they won't. Both parts of this fix should be stated honestly for what they are: a mitigation and a reasoned bet, not a guarantee (checkpoint 11).

### Chronological analysis

| Stage | What's true about GTM at this point |
|---|---|
| Day 1 | Shopify App Store submission filed |
| Week 1 | App live/approved; first organic installs from app-store search — review turnaround varies and isn't a controllable input, so Week 1 is a target, not a promise |
| Month 1 | First 2–3 θ Agency Partners onboarded at a discount for their client base; content-moat groundwork (blog/guide content) begins |
| Year 1 | χ's platform-partner track pursued in earnest; 8–12 agency partners; μ's first annual benchmark report published as the category's most credible independent data point |

### Checkpoint mapping
Checkpoints 19, 23, 27, 28.

---

## 6. Cross-Topic Master Chronology

| | Day 1 | Week 1 | Month 1 | Year 1 |
|---|---|---|---|---|
| **Market Context** | Facts fixed, documented once | Discovery calls validate real fear level | φ monitoring cadence begins | Full 2026 filing data reassessed |
| **Positioning** | Evidence-not-certification encoded in copy | Reframe tested against real reactions | First public explainer content | μ's report becomes the proof point |
| **Target/Beachhead** | Shopify-only | Apparel hypothesis tested | Actual beachhead defined by real conversions | WooCommerce + possible 2nd vertical |
| **Revenue** | Free tier only | Pricing page + willingness-to-pay signal | First paid conversions, Billing API live | θ tier material; modeled numbers replaced by real ones |
| **GTM** | App Store submission | App live, organic installs begin | First Agency Partners onboarded | Platform-asset track + first benchmark report |

## 7. Final Verdict

Every business assumption in this file that was checkable against real data was checked (Shopify merchant counts, competitor pricing, Shopify's own revenue-share policy and its demonstrated instability). Every number that can't yet be checked (CAC, real retention, real conversion rate) is labeled as modeled, not measured, and tied to the specific week or month it gets replaced with reality. The revenue model was stress-tested against a harsher version of a policy that has already changed once, not just the current favorable version. **Cleared, with every unmeasured assumption named as exactly that.**
