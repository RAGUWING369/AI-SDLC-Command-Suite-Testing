# Success Metrics — ShopNest
**Phase:** 01 — Ideation
**Generated:** 2026-05-04
**Status:** Draft — Awaiting Human Gate Approval

---

## Desired Outcome

> **Achieve 100 active paying seller subscriptions within 6 months of public launch,
> with a monthly churn rate below 5% — proving that ShopNest delivers enough ongoing
> value to retain sellers beyond the first invoice.**

This is the primary outcome signal that validates product-market fit for the seller side.
Buyer-side validation is secondary but critical: sellers will churn if buyers don't transact.

---

## Business KPIs

| Metric | Baseline | Target | Timeline | Measurement Method |
|--------|----------|--------|----------|--------------------|
| Active Paying Sellers (subscriptions) | 0 | 100 | 6 months post-launch | Subscription DB (Razorpay Subscriptions + ShopNest DB) |
| Monthly Recurring Revenue (MRR) | ₹0 | ₹1,99,900 (100 × ₹1,999) | 6 months post-launch | Razorpay billing records |
| Seller Monthly Churn Rate | — | < 5% | From Month 3 post-launch | Subscription cancellations / active subscriptions |
| Seller Lifetime Value (LTV) | — | > ₹23,988 (12-month LTV at ₹1,999/month, <5% monthly churn) | Month 12 | Cohort analysis |
| AWS Infrastructure Cost per Seller | — | < ₹2,000/seller/month at 100 sellers | Month 6 | AWS Cost Explorer |
| Gross Margin | — | > 15% at 100 sellers; > 50% at 500 sellers | Month 6 / Month 18 | P&L: MRR minus AWS + payment processing costs |

---

## User KPIs — Seller Side

| Metric | Baseline | Target | Timeline | Measurement Method |
|--------|----------|--------|----------|--------------------|
| Seller Store Setup Completion Rate | 0% | > 80% | Month 1 post-launch | % of registered sellers who complete setup wizard AND list ≥ 5 products |
| Time to First Product Listed | — | < 30 minutes from signup | Month 1 | Session timestamps in app analytics |
| Seller Activation Rate | 0% | > 70% | Month 2 | % of paid sellers who receive ≥ 1 confirmed order within 30 days of going live |
| Seller Order Fulfillment Rate | — | > 95% of orders marked Shipped within 48 hours | Month 3 | Order status timestamps |
| Seller NPS (Net Promoter Score) | — | > 40 | Month 3 post-launch | Monthly in-app NPS survey |

---

## User KPIs — Buyer Side

| Metric | Baseline | Target | Timeline | Measurement Method |
|--------|----------|--------|----------|--------------------|
| Cart-to-Order Conversion Rate | — | > 3% [Industry benchmark: 2–4% for mobile e-commerce in India — not project-specific] | Month 3 post-launch | Orders / unique sessions with cart activity |
| Guest Checkout Completion Rate | — | > 60% of initiated guest checkout sessions | Month 3 | Checkout funnel analytics |
| Registered Buyer Repeat Purchase Rate | — | > 25% within 30 days of first purchase | Month 6 | Order history per buyer account |
| Buyer Order Tracking Engagement | — | > 50% of buyers view order status at least once post-purchase | Month 3 | Page view analytics on order tracking page |
| Payment Success Rate (Razorpay) | — | > 97% | From launch | Razorpay dashboard + webhook success/failure logs |

---

## Technical KPIs

| Metric | Baseline | Target | Timeline | Measurement Method |
|--------|----------|--------|----------|--------------------|
| API P95 Response Time (all endpoints) | — | < 200ms | From launch | AWS CloudWatch / APM tool |
| Product Page LCP (Largest Contentful Paint) | — | < 2.5s on mobile (4G) | From launch | Core Web Vitals (Google PageSpeed Insights / Vercel Analytics) |
| Platform Uptime | — | ≥ 99.9% (< 8.7 hours downtime/year) | Ongoing from launch | AWS CloudWatch uptime alarm |
| Product Image Load Time (via CloudFront CDN) | — | < 1s for images ≤ 500KB | From launch | CloudFront access logs + browser performance timing |
| Checkout Error Rate (Razorpay failures + app errors) | — | < 1% of checkout attempts | From launch | Razorpay webhook failure rate + application error logs |
| Database Query P95 Latency | — | < 50ms | From launch | RDS Performance Insights |
| Concurrent Users Supported (load test) | — | ≥ 1,000 without degradation | Pre-launch (Phase 9 Testing) | Load test tooling (Locust or k6) |

---

## Leading Indicators

Early signals that ShopNest is on track — measurable before full KPIs are available.

| Indicator | Target | When to Measure | Why It Matters |
|-----------|--------|-----------------|----------------|
| Beta seller sign-ups (pre-launch waitlist) | ≥ 25 sellers | 4 weeks pre-launch | Validates demand before public launch; provides onboarding test cohort |
| First seller completes full store setup | ≥ 1 | Week 1 post-launch | Proves onboarding flow is functional end-to-end |
| First completed end-to-end order (seller lists → buyer purchases → seller ships) | ≥ 1 | Week 2 post-launch | Proves the core transaction loop works |
| ≥ 10 active sellers (products listed, at least 1 order received) | 10 sellers | Month 1 post-launch | Indicates sellers find enough value to stay active |
| ≥ 20 paying sellers (subscription charged) | 20 sellers | Month 2 post-launch | First real revenue signal; validates price point |
| Seller support ticket volume per seller | < 2 tickets/seller/month | Month 2 | High ticket volume signals onboarding gaps — fix before scaling |

---

## KPI Review Cadence

| Frequency | Metrics Reviewed | Owner |
|-----------|-----------------|-------|
| Weekly | Seller signup count, product listings added, orders placed, checkout error rate | Full-stack lead |
| Monthly | MRR, churn rate, seller NPS, AWS cost per seller, conversion rate | Product + lead dev |
| Quarterly | LTV, gross margin, SOM progress, technical KPI trend | All stakeholders |

---

*Sources: CLAUDE.md NFR targets; gap scan answers 2026-05-04*
*Industry benchmarks explicitly labelled — not project-specific. Confirm targets with team before Phase 3 (PRD).*
*See `docs/assumptions/01-ideation-assumptions.md` for full inference log.*
