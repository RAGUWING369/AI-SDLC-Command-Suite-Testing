# Ideation — Assumption Log
**Phase:** 01 — Ideation
**Agent:** 01_ideation_agent.md
**Generated:** 2026-05-04
**Session:** ShopNest Phase 1 — India-only multi-vendor SaaS marketplace

---

## Tier 3 Inferences Made This Phase

| ID | Inference | Basis | Confidence | Must Validate Before Phase |
|----|-----------|-------|------------|---------------------------|
| A-01-001 | Current dominant seller workaround is WhatsApp Business + Excel/Google Sheets (not a competitor SaaS product) | India SME e-commerce behavioural pattern; consistent with JioMart/Instagram commerce research | High | Phase 2 (Requirements) — validate via 3+ seller interviews before requirements are finalised |
| A-01-002 | >70% of ShopNest buyer traffic will be mobile (smartphone) | India internet traffic composition (TRAI 2024: 78% of internet sessions are mobile); Jio-driven smartphone penetration in Tier 1–2 cities | High | Phase 5 (UX Design) — confirm via analytics post-beta |
| A-01-003 | Razorpay webhook-based payment confirmation is the correct integration pattern for order status update | Razorpay API documentation; standard payment gateway integration pattern | High | Phase 7 (Implementation) — Razorpay sandbox testing |
| A-01-004 | 63 million total Indian MSMEs is the correct figure for market sizing | Ministry of MSME Annual Report 2023 | High (official source) | Annual — refresh before investor reporting |
| A-01-005 | 60% of Indian MSMEs sell physical products (excl. pure service businesses) | Industry benchmark — not project-specific. Source: MSME sector composition estimates | Medium | Phase 1 only — does not affect downstream engineering |
| A-01-006 | 40% of Indian MSMEs have sufficient smartphone/internet to use a SaaS platform | TRAI 2024 data on internet penetration by business sector [benchmark] | Medium (likely understated given Jio penetration) | Phase 1 only — does not affect downstream engineering |
| A-01-007 | 25% of internet-capable MSMEs fall in the 50–500 SKU range (ShopNest's target product complexity) | Industry pattern for SME physical retail product depth | Low-Medium — highest risk SAM assumption | Phase 2 (Requirements) — validate via seller intake survey |
| A-01-008 | Amazon India commission range is 15–40% depending on category | Amazon India Seller Central fee schedule (publicly published) | High | Accept for competitive analysis — verify via Amazon seller portal if used in investor materials |
| A-01-009 | Flipkart commission range is 12–30% by category | Flipkart Seller Hub fee schedule (publicly published) | High | Accept for competitive analysis |
| A-01-010 | Shopify pricing in USD at ~$29–$79/month (Basic to Shopify plan) | Shopify public pricing page as of early 2026 | High | Accept for competitive analysis — verify current pricing at time of use |
| A-01-011 | GST compliance (GSTIN registration, GST-compliant invoicing for subscriptions) is a legal requirement for ShopNest as an Indian SaaS business | Indian GST Act applies to SaaS businesses above the registration threshold (₹20L/year revenue) | High (legal certainty) | Pre-launch — confirm registration threshold and invoice format with a CA before first paying seller |
| A-01-012 | Indian e-commerce marketplace rules (IT Act, Consumer Protection (E-Commerce) Rules 2020) apply to ShopNest as a multi-vendor marketplace | Consumer Protection (E-Commerce) Rules 2020 cover marketplace e-commerce entities | High (legal certainty) | Pre-launch — legal review mandatory before public launch |
| A-01-013 | Mobile cart-to-order conversion rate benchmark of 2–4% for India mobile e-commerce | E-commerce industry benchmark [not project-specific — source: Barilliance, Statista] | Medium | Month 3 post-launch — measure actual ShopNest conversion and set project-specific baseline |
| A-01-014 | Monthly SaaS churn < 5% is an achievable early-stage benchmark for a B2B SaaS product with good onboarding | Early-stage SaaS industry benchmark [not project-specific — source: ChartMogul SaaS Benchmarks] | Medium | Month 3 post-launch — validate from first seller cohort |
| A-01-015 | ~50,000 active Instagram/WhatsApp fashion and lifestyle sellers in Tier 1–2 Indian cities fit the 50–500 SKU profile (beachhead segment) | Tier 3 inference based on social commerce activity data for India [not project-specific] | Low-Medium | Phase 6 (Task Breakdown) — run a structured beta seller recruitment campaign to measure reachable pool |
| A-01-016 | ₹1,999/month is an appropriate mid-market subscription price for Indian SME sellers | India SaaS pricing benchmarks (Zoho, Freshworks India-market products at similar complexity) [not validated with ShopNest's target sellers] | **Low — highest business risk assumption** | Before launch — mandatory pricing validation with 10–15 beta seller interviews in Q3 2026 |
| A-01-017 | AWS cost at 100 sellers is approximately ₹1,66,000/month (~$2,000) — matching the declared budget | Derived from CLAUDE.md budget constraint; confirmed against ECS Fargate + RDS + ElastiCache + CloudFront rough sizing for 1,000 DAU | Medium | Phase 11 (CI/CD) and Phase 12 (Deployment) — validate with actual AWS cost calculator before first paying seller |
| A-01-018 | USD/INR rate of ₹83.5/$1 used for currency conversion in market sizing | Approximate market rate as of May 2026 | Medium | Phase 1 only — use current rate for investor reporting |

---

## Open Flags (Tier 2 — Unconfirmed Suggestions)

| Flag ID | Suggestion Made | Location in Artifact | Status |
|---------|----------------|----------------------|--------|
| F-01-001 | Competitive set should include Amazon India, Flipkart, Shopify, WooCommerce, and Do Nothing | Confirmed by user during gap scan (Tier 1 answered) | ✅ Confirmed |
| F-01-002 | Subscription pricing at ₹1,999/month | MARKET-SIZING.md, SUCCESS-METRICS.md | ⚠️ Pending — validate with beta sellers before launch |
| F-01-003 | Beachhead segment: Indian fashion/lifestyle Instagram sellers | COMPETITIVE-ANALYSIS.md, PROJECT-CONCEPT.md | Pending confirmation from team |

---

## Tier 1 Questions Answered This Session (Gap Scan)

| Question | Answer | Recorded In |
|----------|--------|-------------|
| Target geography | India only | All artifacts |
| Platform revenue model | Monthly SaaS subscription from sellers | PROJECT-CONCEPT.md, MARKET-SIZING.md, SUCCESS-METRICS.md |
| Payment gateway | Razorpay (INR, UPI, NetBanking) | PROJECT-CONCEPT.md, FEASIBILITY-REPORT.md |
| Multi-vendor or single seller | Multi-vendor marketplace | PROJECT-CONCEPT.md, FEASIBILITY-REPORT.md, STAKEHOLDER-MAP.md |
| Guest checkout in MVP | Yes | PROJECT-CONCEPT.md (In Scope) |
| Competitor set | Amazon India, Flipkart, Shopify, WooCommerce, Do Nothing | COMPETITIVE-ANALYSIS.md |

---

## Resolution Log

| ID | Original Assumption | Resolution | Resolved By | Date |
|----|--------------------|-----------|-----------:|------|
| *(No assumptions have been resolved or corrected yet — this phase is complete as of 2026-05-04)* | | | | |
