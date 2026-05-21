# Market Sizing — ShopNest
**Phase:** 01 — Ideation
**Generated:** 2026-05-04
**Geography:** India only
**Unit of Analysis:** SME Seller subscriptions (SaaS seats)
**Status:** Draft — Awaiting Human Gate Approval

---

## Methodology

All sizing is **bottom-up** (count units, then multiply by price) — not top-down percentage slicing of a reported market figure. Industry benchmarks are explicitly labelled and must be validated before Series A fundraising or board-level planning.

---

## TAM — Total Addressable Market

**Definition:** All Indian businesses that sell physical products and could benefit from a hosted e-commerce storefront with India-native payment checkout.

| Input | Value | Source |
|-------|-------|--------|
| Total Indian MSMEs | ~63 million | Ministry of MSME Annual Report 2023 [Industry benchmark] |
| % actively selling physical products (excl. pure service businesses) | ~60% | [Industry benchmark — not project-specific] |
| Eligible MSME universe | ~37.8 million businesses | Derived |
| % with smartphone + internet access sufficient to use a SaaS platform | ~40% | [Industry benchmark based on TRAI 2024 data — not project-specific] |
| TAM seller count | **~15.1 million sellers** | Derived |
| Indicative subscription price | ₹1,999/month | Mid-market India SaaS benchmark — NOT yet validated [Tier 2 — confirm via beta interviews] |
| **TAM (Annual)** | **~₹3.62 lakh crore/year** (~$43B/year) | Derived |

> **Note:** TAM represents the theoretical ceiling, not a realistic capture target. It includes all product-selling MSMEs regardless of category, technical readiness, or competitive context.

---

## SAM — Serviceable Addressable Market

**Definition:** TAM filtered to the segment ShopNest can actually reach: Indian SMEs with 50–500 SKUs, smartphone-literate, operating in Tier 1–2 cities or urban Tier 3, already selling informally online (Instagram/WhatsApp) or actively seeking to go digital.

| Filter Applied | Rationale | Retention % | Seller Count |
|----------------|-----------|------------|--------------|
| TAM | All product-selling, internet-capable MSMEs | 100% | 15.1M |
| 50–500 SKU range | ShopNest MVP product management designed for this range | ~25% | 3.78M |
| Tier 1–2 city concentration or urban Tier 3 with digital literacy | Geographic and infrastructure fit | ~30% | 1.13M |
| Already selling informally online OR actively seeking e-commerce solution | Demand signal qualifier | ~75% | 0.85M |
| **SAM seller count** | | | **~850,000 sellers** |
| Annual subscription value | ₹1,999 × 12 months | | ₹23,988/seller/year |
| **SAM (Annual)** | | | **~₹2,039 crore/year** (~$244M/year) |

> **Highest-risk SAM assumption:** The 50–500 SKU filter retaining 25% of internet-capable MSMEs is an informed estimate — not validated data. [Industry benchmark — validate with a structured seller survey in Q3 2026 before Series A.]

---

## SOM — Serviceable Obtainable Market

**Definition:** Realistic seller count ShopNest can acquire and retain over a 12–36 month horizon, given a 3-person team, ₹2,000/month AWS budget, India-only focus, and no declared paid marketing budget.

**SOM Rationale:** ShopNest's primary acquisition channel in Year 1 is referral-driven (beachhead sellers in fashion/lifestyle communities), supplemented by content marketing and SEO. No paid acquisition budget declared.

| Period | Sellers | MRR | Annual Revenue |
|--------|---------|-----|----------------|
| Month 6 post-launch (MVP milestone) | 100 | ₹1,99,900 | ₹11,99,400 |
| Month 12 post-launch (Year 1 end) | 500 | ₹9,99,500 | ₹59,97,000 |
| Month 24 post-launch (Year 2 end) | 2,000 | ₹39,98,000 | ₹2.4 crore |
| Month 36 post-launch (Year 3 end) | 5,000 | ₹99,95,000 | ₹11.99 crore |

**SOM as % of SAM:** 5,000 sellers / 850,000 SAM = **0.59%** of SAM at Year 3. Conservative and defensible.

---

## Sensitivity Analysis — Base / Best / Worst Case

| Scenario | Assumption Change | Sellers at Month 12 | MRR at Month 12 | Annual Run Rate |
|----------|-------------------|--------------------|-----------------|-|
| **Best Case** | Viral adoption via social commerce; price validated at ₹2,499/month | 1,500 | ₹37,48,500 | ₹4.5 crore |
| **Base Case** | Referral-led growth; price at ₹1,999/month | 500 | ₹9,99,500 | ₹1.2 crore |
| **Worst Case** | Slow seller acquisition (CAC too high); 10%+ monthly churn | 150 | ₹2,99,850 | ₹35.98 lakh |

**Worst Case Trigger:** If Month 3 active sellers < 30 or monthly churn > 8%, immediately investigate onboarding and pricing. Do not scale before fixing retention.

---

## Key Assumptions Register

| ID | Assumption | Source | Risk Level | Validation Action |
|----|-----------|--------|-----------|-------------------|
| MA-001 | 63M total Indian MSMEs | Ministry of MSME 2023 [Industry benchmark] | Low | Accept for sizing; refresh annually |
| MA-002 | 60% of MSMEs sell physical products | [Industry benchmark — not project-specific] | Medium | Refine via seller survey |
| MA-003 | 40% of MSMEs have sufficient smartphone/internet to use SaaS | TRAI 2024 data [Industry benchmark] | Medium | Conservative; likely understated given Jio penetration |
| MA-004 | 25% of internet-capable MSMEs fall in 50–500 SKU range | [Tier 3 inference — industry pattern for SME product complexity] | High | Validate via beta seller intake form |
| MA-005 | ₹1,999/month subscription price acceptable to target sellers | [Mid-market India SaaS benchmark — NOT validated] | **Very High** | Validate with 10–15 beta seller price interviews before launch |
| MA-006 | Monthly churn < 5% from Month 3 | [Early-stage SaaS benchmark — not project-specific] | High | Track from first paying cohort; adjust onboarding if > 5% |
| MA-007 | Primary acquisition via referral (no paid marketing) | CLAUDE.md — no marketing budget declared | Medium | If Month 6 sellers < 50, reconsider acquisition strategy |

---

## Infrastructure Cost Scaling Model

| Active Sellers | Estimated AWS Monthly Cost | Cost/Seller | Notes |
|----------------|--------------------------|------------|-------|
| 100 | ₹1,66,000 (~$2,000) | ₹1,660 | Budget ceiling for MVP |
| 500 | ₹2,50,000–₹3,50,000 | ₹500–₹700 | Sub-linear scaling via managed services |
| 2,000 | ₹6,00,000–₹8,00,000 | ₹300–₹400 | Reserved instances + autoscaling |
| 5,000 | ₹12,00,000–₹15,00,000 | ₹240–₹300 | CDN and S3 dominant cost drivers at scale |

> At 500 sellers, gross margin exceeds 50%. At 100 sellers, margin is thin (~17%) but positive. Do not expand team or marketing spend before 100-seller milestone.

---

*All figures in INR. USD equivalents at ₹83.5/$1 (approximate — May 2026).*
*Industry benchmarks labelled — not project-specific data. Validate before investor reporting.*
*See `docs/assumptions/01-ideation-assumptions.md` for full inference log.*
