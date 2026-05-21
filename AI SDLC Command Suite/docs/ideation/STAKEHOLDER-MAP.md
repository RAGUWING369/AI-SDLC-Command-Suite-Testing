# Stakeholder Map — ShopNest
**Phase:** 01 — Ideation
**Generated:** 2026-05-04
**Framework:** BABOK v3, Section 3.2 — Plan Stakeholder Engagement
**Status:** Draft — Awaiting Human Gate Approval

---

## Stakeholder Registry

| Stakeholder | Type | Primary Job / Interest | Influence | Impact if Not Engaged | Engagement Strategy | Review Cadence |
|-------------|------|----------------------|-----------|----------------------|--------------------|----|
| **SME Seller / Store Owner** | Primary User + Economic Buyer | List products, manage orders, receive payments, track revenue. Subscribes monthly — they are the primary revenue source for ShopNest. | **High** — churn = direct revenue loss | Sellers churn; platform has no revenue; product-market fit never achieved | Beta seller program (10–15 sellers pre-launch); structured onboarding interviews; in-app feedback prompts; monthly seller NPS survey | Per sprint (pre-launch); monthly (post-launch) |
| **Indian Online Buyer** | Primary User | Browse products across sellers, add to cart, checkout via UPI/card, track orders. Can be guest or registered. | **High** — low buyer adoption = sellers don't get orders = seller churn | No transactions; sellers see zero ROI and churn; marketplace remains empty | Usability testing sessions on mobile (375px); A/B testing on checkout flow; post-purchase satisfaction survey | Per sprint (pre-launch); ongoing (post-launch via in-app) |
| **Platform Admin (ShopNest dev team)** | Operator | Approve/suspend seller accounts, moderate product listings, resolve billing issues, monitor platform health, manage deployments. | **High** — internal team controls all platform decisions | Unmoderated platform abuse; seller billing disputes unresolved; outages undetected | Weekly internal sprint demos; admin panel built as first-class tool (not afterthought); on-call rotation defined pre-launch | Daily standup; weekly sprint review |
| **Razorpay** | Technical / Payment Partner | Process buyer payments (UPI, cards, NetBanking, EMI) and seller subscription billing. Handle chargebacks and payment compliance. | **Medium** — platform cannot accept payments without Razorpay integration | Checkout broken; no revenue; no subscription billing | Integration testing in Razorpay sandbox before launch; webhook reliability testing; Razorpay support escalation path documented | Per integration milestone; post-launch monthly review |
| **AWS** | Infrastructure Partner | Provide compute (ECS Fargate), database (RDS PostgreSQL), cache (ElastiCache Redis), object storage (S3), CDN (CloudFront). | **Medium** — infrastructure outages = platform downtime | Platform unavailable; seller orders lost; buyer trust broken | AWS Cost Explorer monitoring (alert at 80% budget); CloudWatch dashboards; Reserved capacity planning for post-MVP scale | Monthly cost review; per-incident |
| **Development Team (3 devs)** | Delivery Stakeholder | Design, build, test, and deploy all platform features within 6.5-month MVP timeline. 1 full-stack lead, 1 frontend, 1 backend. | **High** — delivery risk is the #1 schedule threat | Missed deadline; scope creep; technical debt accumulation | Sprint planning (2-week cycles); architecture decision records (ADRs) documented; code review mandatory before merge; no single-person knowledge bottlenecks | Daily standup; bi-weekly sprint ceremony |
| **GST / Regulatory (GSTIN, MCA)** | Regulator | Ensure ShopNest as a SaaS business issues GST-compliant invoices for subscriptions. E-commerce rules (IT Act, Consumer Protection Act) apply to marketplace operations. | **Medium** — non-compliance = legal exposure | Legal penalties; platform shutdown risk; seller distrust | GST invoice generation for ShopNest subscriptions built before first paying seller; consult CA for e-commerce marketplace compliance pre-launch | Pre-launch legal review; annually thereafter |
| **Logistics / Courier Partners** (post-MVP) | Indirect Stakeholder | Sellers use third-party couriers (DTDC, Delhivery, Shiprocket) for physical delivery. ShopNest does not manage logistics in MVP. | **Low (MVP)** — seller manages own logistics | Sellers frustrated by manual AWB entry; post-MVP integration opportunity | Note as post-MVP integration opportunity; seller dashboard includes free-form AWB number field for MVP | Post-MVP scoping session |

---

## Coverage Verification

| Stakeholder Category | Covered? | Notes |
|---------------------|----------|-------|
| End Users (Buyers) | ✅ Yes | Primary User — Indian Online Buyer |
| Clients / Economic Buyers | ✅ Yes | SME Seller / Store Owner (subscription payer) |
| Operators / Admins | ✅ Yes | Platform Admin (ShopNest dev team) |
| Developers / QA | ✅ Yes | Development Team (3 devs) |
| Technical Partners | ✅ Yes | Razorpay, AWS |
| Regulators / Compliance | ✅ Yes | GST / Regulatory (GSTIN, MCA) |
| Indirect / Downstream | ✅ Yes | Logistics/Courier Partners (noted as post-MVP) |

---

## Engagement Priority Matrix

```
High Influence
     │
     │  SME Seller ●         Platform Admin ●
     │
     │  Indian Buyer ●        Dev Team ●
     │
     │              Razorpay ●    AWS ●
     │
     │  GST/Regulatory ●
     │
     └─────────────────────────────────────────── High Impact if Not Engaged
```

**Top 3 Engagement Priorities:**
1. **SME Seller** — Revenue source; churn = platform death. Prioritise beta program and onboarding quality.
2. **Indian Buyer** — Adoption driver; poor mobile UX = cart abandonment = seller dissatisfaction.
3. **Platform Admin (Dev Team)** — The team is both builder and operator in MVP phase. Admin tooling must be production-ready at launch.

---

*Sources: CLAUDE.md; gap scan answers 2026-05-04*
*Regulatory notes are Tier 3 inferences based on Indian e-commerce law — confirm with legal counsel pre-launch.*
*See `docs/assumptions/01-ideation-assumptions.md` for full inference log.*
