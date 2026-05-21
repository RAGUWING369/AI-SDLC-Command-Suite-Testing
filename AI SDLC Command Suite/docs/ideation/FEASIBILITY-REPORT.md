# Feasibility Report — ShopNest
**Phase:** 01 — Ideation
**Generated:** 2026-05-04
**Status:** Draft — Awaiting Human Gate Approval

---

## Executive Summary

ShopNest is assessed as **GO** across all four feasibility dimensions, with Yellow (proceed-with-mitigation) flags on Economic, Operational, and Schedule dimensions. No Red conditions exist. Key risks are seller acquisition cost (economic), team capacity for post-launch support (operational), and MVP scope discipline (schedule).

---

## Feasibility Matrix

### Dimension 1 — Technical Feasibility
**Rating: 🟢 Green — Proceed**

| Attribute | Assessment |
|-----------|-----------|
| **Stack maturity** | Next.js 14 (App Router), Django 5.x + DRF 3.15, PostgreSQL 16, Redis 7 — all production-proven, with large ecosystems and long-term support |
| **Team alignment** | Declared team skills match the target stack. No technology mismatch identified. |
| **Multi-vendor complexity** | Multi-vendor data isolation (products, orders, revenue scoped per seller) adds schema complexity but is a well-understood Django ORM pattern using ForeignKey + UUID PKs. No novel engineering required. |
| **Razorpay integration** | Razorpay provides a well-documented Python SDK, sandbox environment, and webhook support for payment confirmation. Guest checkout flows are supported. Subscription billing for sellers (ShopNest's own SaaS billing) is available via Razorpay Subscriptions API. |
| **AWS infrastructure** | ECS Fargate (containerised, no EC2 management), RDS for PostgreSQL, ElastiCache for Redis, S3 for product images, CloudFront for CDN delivery. All managed services — appropriate for a 3-person team without dedicated DevOps. |
| **SSR requirement** | Next.js App Router with React Server Components provides SSR for product and category pages natively. No additional framework required. |
| **UUID primary keys** | Declared in CLAUDE.md as a project standard. Straightforward with Django's UUIDField and PostgreSQL's native UUID type. |

**Key Risks:**
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Razorpay webhook delivery failure during checkout | Low | High | Idempotent payment confirmation endpoint; order status state machine with explicit "payment pending" state |
| Multi-vendor data isolation bug (seller A seeing seller B data) | Low | Critical | Row-level security at ORM layer; seller_id FK enforced on all queries; automated tests covering cross-tenant data access |
| AWS ECS cold-start latency during low-traffic periods | Medium | Low | Set minimum task count = 1; use Fargate Spot for dev/staging only |

---

### Dimension 2 — Economic Feasibility
**Rating: 🟡 Yellow — Proceed with Mitigation**

| Attribute | Assessment |
|-----------|-----------|
| **Revenue model** | Monthly SaaS subscription per seller. Flat rate — no per-transaction commission. |
| **Indicative pricing** | ₹1,999/month (mid-market India SaaS benchmark — not yet validated with target sellers) [Industry benchmark — confirm via beta seller interviews] |
| **Break-even seller count** | AWS budget ₹1,66,000/month (~$2,000). At ₹1,999/month per seller, break-even = ~83 sellers. Target: 100 sellers at Month 6 post-launch. |
| **Unit economics at 100 sellers** | MRR = ₹1,99,900. AWS cost = ₹1,66,000. Gross margin = ₹33,900/month (~17%). Thin but positive at MVP scale. |
| **Unit economics at 500 sellers** | MRR = ₹9,99,500. AWS cost estimates ₹3,00,000–₹4,00,000 (infrastructure scales sub-linearly). Gross margin = ~60%. |
| **Market size** | SAM = ~950,000 eligible Indian SME sellers. SOM (Year 1) = 500 sellers = 0.05% SAM. Highly achievable in principle; constrained by CAC and team capacity. |

**Key Risks:**
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Seller acquisition cost (CAC) unknown — no marketing budget declared | High | High | Prioritise referral-driven acquisition via beachhead segment (fashion/lifestyle sellers with social media audiences); measure CAC from first beta cohort |
| Subscription price point wrong — sellers unwilling to pay ₹1,999/month | Medium | High | Validate pricing with 10–15 beta sellers before launch; offer a 30-day free trial to reduce signup friction |
| High early churn (>10%/month) before product-market fit | Medium | High | In-app onboarding checklist; seller success milestone emails; track "first order received" as key activation event |
| AWS cost overrun if seller product image storage grows faster than modelled | Low-Medium | Medium | S3 lifecycle policies; image compression on upload; CDN caching to reduce origin hits |

**Mitigation Actions:**
1. Conduct 10–15 structured pricing interviews with target sellers before launch (Q3 2026)
2. Set up AWS Cost Explorer alerts at 80% of monthly budget
3. Track CAC and LTV from the first 20 sellers; adjust acquisition strategy at Month 2

---

### Dimension 3 — Operational Feasibility
**Rating: 🟡 Yellow — Proceed with Mitigation**

| Attribute | Assessment |
|-----------|-----------|
| **Team capacity** | 3 developers: 1 full-stack lead, 1 frontend, 1 backend. No dedicated DevOps, no dedicated customer support. |
| **Infrastructure management** | AWS managed services (Fargate, RDS, ElastiCache, S3) significantly reduce operational overhead. No server patching required. |
| **Seller support load** | Post-launch seller support (onboarding issues, billing questions, product listing help) will land on the dev team until a support function exists. High risk if seller base grows faster than support capacity. |
| **Payment dispute handling** | Razorpay handles chargebacks and payment disputes. ShopNest's exposure is limited to subscription billing disputes. |
| **GST compliance** | E-commerce platforms in India must issue GST-compliant invoices for subscriptions. Django can generate invoices programmatically. [Inferred: GST registration required for ShopNest as a SaaS business — confirm with legal/CA before launch] |
| **Content moderation** | Multi-vendor marketplace creates risk of prohibited product listings. Admin panel must include seller suspension and product removal capability. |

**Key Risks:**
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Dev team overwhelmed by seller support post-launch | High | Medium | Build self-serve onboarding (guided wizard, in-app help, FAQ); delay launch until seller success documentation is complete |
| Platform abuse (counterfeit/prohibited product listings) | Medium | High | Seller verification step at onboarding (Aadhaar/GSTIN verification — post-MVP; manual review for MVP); admin panel for product moderation |
| No monitoring/alerting capability at launch | Medium | High | Implement CloudWatch alarms + PagerDuty (or simple email alerts) before launch; define on-call rotation among 3 devs |

**Mitigation Actions:**
1. Complete seller onboarding documentation and FAQ before public launch
2. Configure CloudWatch dashboards and basic alerts as part of Phase 11 (CI/CD) and Phase 13 (Monitoring)
3. Define a content moderation policy and admin review SLA before accepting first external sellers

---

### Dimension 4 — Schedule Feasibility
**Rating: 🟡 Yellow — Proceed with Mitigation**

| Attribute | Assessment |
|-----------|-----------|
| **Available time** | 2026-05-04 to 2026-10-31 = ~26 weeks (6.5 months) |
| **Team velocity** | 3 developers; assuming 2-week sprints = 13 sprints available |
| **Scope complexity** | Multi-vendor marketplace + guest checkout + Razorpay subscription billing + SSR product pages + seller dashboard + admin panel = significant scope for a 3-person team |
| **Risk factor** | Phase 2 (Requirements) through Phase 6 (Task Breakdown) will consume 3–4 weeks, leaving ~22 weeks for implementation. Tight but achievable with strict scope control. |
| **Buffer** | Zero buffer for post-MVP features. Any scope creep to wishlist/reviews/analytics will push MVP past deadline. |

**Key Risks:**
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Scope creep (non-MVP features requested mid-build) | High | High | CLAUDE.md Out-of-Scope list is the authority; all feature requests evaluated against it before any work begins |
| Razorpay Subscriptions API integration complexity underestimated | Medium | Medium | Spike on Razorpay Subscriptions in Phase 7 Sprint 1; resolve before committing to billing architecture |
| Frontend/backend integration delays (API contract mismatches) | Medium | Medium | Define OpenAPI spec in Phase 4 (Architecture) before any implementation begins; enforce contract-first development |
| Team member unavailability (illness, leave) | Low-Medium | High | Document all architecture decisions in ADRs; ensure no single-person knowledge bottlenecks |

**Mitigation Actions:**
1. Enforce strict scope gate: no feature begins implementation until it appears in the approved In-Scope list
2. Spike Razorpay Subscriptions integration in Week 1 of implementation
3. Define API contracts (OpenAPI 3.0 spec) before any frontend development begins

---

## Go / No-Go Recommendation

| Dimension | Rating | Decision |
|-----------|--------|----------|
| Technical Feasibility | 🟢 Green | GO |
| Economic Feasibility | 🟡 Yellow | GO with mitigation |
| Operational Feasibility | 🟡 Yellow | GO with mitigation |
| Schedule Feasibility | 🟡 Yellow | GO with mitigation |
| **Overall** | **🟡 Yellow** | **GO — proceed with stated mitigations** |

**Recommendation:** Proceed to Phase 2 (Requirements). No Red conditions exist. The three Yellow flags share a common root cause: a 3-person team executing an ambitious scope under a fixed budget. All three are mitigable with disciplined scope control, early pricing validation, and managed-service infrastructure choices already declared in the stack.

**The single biggest risk to monitor:** Seller acquisition cost and churn. If the first 20 sellers do not activate (list products and receive orders), adjust onboarding — do not add features.

---

*Sources: CLAUDE.md; gap scan answers 2026-05-04*
*Benchmarks and inferences labelled — see `docs/assumptions/01-ideation-assumptions.md`*
