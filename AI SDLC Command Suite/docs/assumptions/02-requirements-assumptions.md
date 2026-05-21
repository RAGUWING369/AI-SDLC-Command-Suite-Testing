# Requirements Engineering — Assumption Log
**Phase:** 02 — Requirements Engineering
**Agent:** 02_requirements_agent.md
**Generated:** 2026-05-04
**Session:** ShopNest Phase 2 — India-only multi-vendor SaaS marketplace

---

## Tier 3 Inferences Made This Phase

| ID | Inference | Basis | Confidence | Must Validate Before Phase |
|----|-----------|-------|------------|---------------------------|
| A-02-001 | Low-stock alert threshold set at 5 units | Common e-commerce platform default (Shopify, WooCommerce default = 5); no project-specific figure provided | High | Phase 9 (Testing) — test FR-PRODUCT-010 fires at exactly 5 units |
| A-02-002 | PostgreSQL full-text search (FTS) is sufficient for product search at MVP scale (≤50,000 products) | Standard choice for Django/PostgreSQL stacks at sub-100K record volumes; Elasticsearch is a post-MVP concern when search quality/volume demands it | High | Phase 9 (Testing) — load test must confirm NFR-PE-005 (P95 ≤ 500ms at 50K products) |
| A-02-003 | AWS Simple Email Service (SES) as the transactional email provider | Derived from declared AWS stack in CLAUDE.md; SES is the natural AWS-native email choice; requires domain verification before launch | High | Phase 7 (Implementation) — SES domain verification and DKIM setup required |
| A-02-004 | Razorpay Payouts API with NEFT/IMPS mode for weekly seller settlement | Razorpay Payouts API is Razorpay's standard disbursement product for Indian bank accounts; NEFT and IMPS are the standard transfer modes for Indian bank accounts | High | Phase 7 (Implementation) — Razorpay Payouts sandbox testing required; ShopNest's Razorpay account needs Payouts feature enabled (separate onboarding) |
| A-02-005 | JWT tokens signed with RS256 (asymmetric) rather than HS256 (symmetric) | RS256 is preferred for multi-service architectures (public key can be shared); more secure than HS256 for a platform with multiple consumer-facing apps (seller dashboard, buyer app) | High | Phase 7 (Implementation) — key pair generation and rotation policy needed |
| A-02-006 | Monday 09:00 IST as the weekly payout execution time | Standard practice for Indian B2B financial transactions (NEFT available 24x7 from Dec 2019, but 09:00 IST Monday is conventional business-day start for batch jobs) | Medium | Phase 7 (Implementation) — confirm with business stakeholder; acceptable to change to a different day/time without rework |
| A-02-007 | Minimum payout amount = ₹1 (Razorpay Payouts API minimum) | Razorpay Payouts API minimum transaction amount for NEFT/IMPS is ₹1 [Razorpay API documentation] | High | Phase 7 (Implementation) — verify against current Razorpay Payouts API documentation |
| A-02-008 | Seller settlement ledger credits the full order amount net of Razorpay payment processing fee (~2% + GST) | Razorpay deducts processing fees from the merchant's collected amount before settlement; ShopNest absorbs this fee as a cost of the zero-commission model (BR-005) | Medium | Phase 7 (Implementation) — confirm exact Razorpay fee structure for ShopNest's account plan |
| A-02-009 | Product image MIME type validation at upload time (server-side) is sufficient; no client-side restriction required for MVP | Server-side MIME validation is the security control; client-side is UX convenience only | High | Phase 10 (Security Review) — confirm no bypass exists for server-side MIME check |
| A-02-010 | Order tracking URL uses HMAC-SHA256 token (derived from order UUID + secret) as the authentication mechanism for guest order access | Standard stateless authentication pattern for guest order tracking; avoids requiring session or account | High | Phase 7 (Implementation) — HMAC secret must be stored securely (env var, not codebase) |
| A-02-011 | Seller bank account details stored encrypted at rest using AWS RDS encryption (AES-256) | AWS RDS encryption at rest is declared in NFR-SEC-002; covers all RDS data including bank account fields | High | Phase 12 (Deployment) — verify RDS encryption is enabled before first seller onboarding |
| A-02-012 | Razorpay Subscriptions API manages recurring seller billing; no custom billing cycle implementation required | Razorpay Subscriptions API handles: invoice generation, automatic retry on failure, webhook events (charged / charge.failed) | High | Phase 7 (Implementation) — Razorpay Subscription plan must be created in Razorpay dashboard before seller onboarding begins |
| A-02-013 | English-only UI for MVP; no i18n framework setup required | Confirmed in Phase 2 gap scan (CON-006); avoids i18n complexity for a 3-person team | High | Phase 7 (Implementation) — hardcoded English strings acceptable; no i18n library needed |
| A-02-014 | Cart data stored in Redis as JSON string keyed by session UUID (guest) or buyer UUID (registered) | Standard Django/Next.js cart implementation pattern; Redis is declared in the stack; 24-hour TTL confirmed in requirements | High | Phase 7 (Implementation) — Redis key naming convention to be agreed in Phase 4 (Architecture) |
| A-02-015 | Audit log stored in a dedicated PostgreSQL table (not a separate service) | Sufficient for MVP volume (≤500K entries/year); ELK/Splunk is post-MVP | High | Phase 9 (Testing) — verify audit log entries written for all NFR-MAINT-005 events |

---

## Open Flags (Tier 2 — Unconfirmed Suggestions Accepted This Phase)

| Flag ID | Suggestion Made | Accepted By | Status |
|---------|----------------|-------------|--------|
| F-02-001 | GST invoicing scope: subscription billing only (not seller-to-buyer) | User (Phase 2 gap scan, 2026-05-04) | ✅ Confirmed |
| F-02-002 | Email-only notifications for MVP (no SMS/WhatsApp) | User (Phase 2 gap scan, 2026-05-04) | ✅ Confirmed |
| F-02-003 | Seller registration data fields as specified | User (Phase 2 gap scan, 2026-05-04) | ✅ Confirmed |
| F-02-004 | Auth: JWT 24h/30d, bcrypt, concurrent sessions, 8+ char password | User (Phase 2 gap scan, 2026-05-04) | ✅ Confirmed |
| F-02-005 | 7-day grace period after subscription payment failure | User (Phase 2 gap scan, 2026-05-04) | ✅ Confirmed |

---

## Tier 1 Questions Answered This Phase (Gap Scan)

| Question | Answer | Impact | Recorded In |
|----------|--------|--------|-------------|
| Seller payout mechanism | Settlement cycle — weekly via Razorpay Payouts API | FR-PAYOUT-001 through FR-PAYOUT-006; BR-006; UC-005 | REQUIREMENTS.md §FR-PAYOUT; BUSINESS-RULES.md BR-006 |
| Order cancellation policy | Buyer can cancel before seller confirms; no returns in MVP | FR-ORDER-003; BR-002; BR-003; UC-006 | REQUIREMENTS.md §FR-ORDER; BUSINESS-RULES.md BR-002, BR-003 |
| Product listing approval | Admin-approved before going live | FR-PRODUCT-003, FR-PRODUCT-004, FR-PRODUCT-005; BR-001 | REQUIREMENTS.md §FR-PRODUCT; BUSINESS-RULES.md BR-001 |
| Subscription plan structure | Single flat plan — all features | FR-SUBSCR-001; simplified billing logic | REQUIREMENTS.md §FR-SUBSCR |

---

## Phase 1 Assumptions Now Resolved or Carried Forward

| Phase 1 ID | Original Assumption | Phase 2 Status | Resolution |
|------------|--------------------|--------------|----|
| A-01-001 | WhatsApp + spreadsheet is the dominant seller workaround | Carried to Phase 9 | Validate via seller usability test (NFR-USA-004) |
| A-01-003 | Razorpay webhook confirmation is the correct pattern | ✅ Confirmed — specified in FR-CHECKOUT-005 | Written into must-have requirement |
| A-01-007 | 25% of internet-capable MSMEs in 50–500 SKU range | Phase 1 only — no Phase 2 impact | Not actionable in requirements |
| A-01-011 | GST compliance required for SaaS billing | ✅ Confirmed — BR-010, FR-SUBSCR-007 | Written into business rule and functional requirement |
| A-01-016 | ₹1,999/month pricing | Still unvalidated — requires beta seller interview | Carried to Phase 9 / pre-launch validation |

---

## Resolution Log

| ID | Original Assumption | Resolution | Resolved By | Date |
|----|--------------------|-----------|-----------:|------|
| A-02-001 | Low-stock threshold = 5 units | Accepted as project standard | Phase 2 agent (Tier 3) | 2026-05-04 |
| A-02-002 | PostgreSQL FTS for MVP search | Accepted; validation deferred to Phase 9 load test | Phase 2 agent (Tier 3) | 2026-05-04 |
