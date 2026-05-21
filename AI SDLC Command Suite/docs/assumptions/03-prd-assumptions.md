# PRD — Assumption Log
**Phase:** 03 — Product Requirements Document
**Agent:** 03_prd_agent.md
**Generated:** 2026-05-05
**Session:** ShopNest Phase 3 — PRD synthesis (multi-vendor India SaaS marketplace)

---

## Tier 3 Inferences Made This Phase

| ID | Inference | Basis | Confidence | Must Validate Before Phase |
|----|-----------|-------|------------|---------------------------|
| A-03-001 | Internal-only analytics (first-party PostgreSQL `analytics_events` table) with no third-party SDK (Mixpanel, Segment, Amplitude) for MVP | 3-developer team + $2,000/month AWS budget makes third-party analytics SaaS an unnecessary cost at MVP scale; first-party approach avoids DPDPA third-party data-sharing complexity; analytics volume at MVP scale (≤100 sellers, ≤1,000 DAU) is trivially handled by PostgreSQL | High | Phase 9 (Testing) — TC-ANLX tests validate instrumentation |
| A-03-002 | Analytics API endpoint responds 202 Accepted and writes to DB asynchronously via Celery to avoid blocking critical path | Standard pattern for high-throughput analytics ingestion; ensures analytics never degrades checkout or product page performance (NFR-PE-001) | High | Phase 7 (Implementation) — Celery task queue must be operational before analytics endpoint is instrumented |
| A-03-003 | SHA-256 hash of search query stored (not raw query text) in `search.query_submitted` event | Protects against PII leakage in search (buyers may search their own names or other PII-adjacent terms); hash enables frequency/duplicate analysis without raw text exposure; consistent with DPDPA data minimization principle | High | Phase 10 (Security Review) — confirm no PII bypass exists |
| A-03-004 | Analytics events older than 2 years (non-financial) purged via Celery beat job; financial-adjacent events retained for 7 years (BR-011) | 7-year retention is required for financial records (BR-011, Companies Act + GST Act); behavioral analytics (page views, cart events, search) are not financial records and 2-year retention is sufficient for operational learning; avoids unbounded database growth | High | Phase 7 (Implementation) — retention purge job must be defined; Phase 10 (Security Review) — confirm retention classification is correct |
| A-03-005 | `user_id` in `analytics_events` is set to NULL (not deleted) on account deletion to preserve aggregate metrics while severing the PII link | Standard DPDPA-compliant erasure pattern for analytics tables; deleting rows would corrupt funnel and cohort metrics; nulling the FK severs the PII link while retaining statistical signal | High | Phase 7 (Implementation) — account deletion flow must trigger `analytics_events` user_id nullification |
| A-03-006 | `cart.abandoned` event detected by a Celery beat hourly scan (not a Redis expiry callback) | Redis does not natively support reliable expiry callbacks at scale; Celery beat hourly scan is more reliable and debuggable; 1-hour detection lag is acceptable for MVP analytics (no real-time abandonment recovery in scope) | High | Phase 7 (Implementation) — Celery beat schedule must include cart abandonment scanner |
| A-03-007 | Product prices stored and transmitted in analytics as integer paise (₹1 = 100 paise) to avoid floating-point precision errors | Standard Indian fintech practice; avoids ₹29.90 represented as 29.899999... in floating-point; consistent with Razorpay API's integer-amount convention (Razorpay requires amounts in paise) | High | Phase 7 (Implementation) — all price fields in analytics events must use paise; PRD and analytics plan are consistent |
| A-03-008 | Feature flags (`ff_payout_live`, `ff_product_admin_approval`, `ff_guest_checkout`) implemented as Django settings/environment variables for MVP, not a dedicated feature flag service (e.g., LaunchDarkly) | A dedicated feature flag service is over-engineered for a 3-developer MVP team with 3 flags; environment variable toggles are sufficient and maintain the same gate semantics; post-MVP flag proliferation would justify a dedicated service | High | Phase 7 (Implementation) — feature flag convention must be documented in CLAUDE.md after Phase 4 |
| A-03-009 | "Why Now" framing in PRD §2 uses 5 market timing factors (UPI ubiquity, social commerce maturity, competitive squeeze, regulatory legitimization, team positioning) as synthesis from Phase 1 artifacts without additional primary research | All 5 factors are grounded in Phase 1 COMPETITIVE-ANALYSIS.md and MARKET-SIZING.md; no new research was conducted in Phase 3; labelled as synthesis, not new claims | High | Phase 1 artifacts already approved — no re-validation needed |
| A-03-010 | "Rejected seller email blocked from re-registration; must contact support to appeal" (Gap G-001 resolution) is implemented as a blacklist check on the `users` table `email` field at registration time, not a separate blacklist table | Simplest implementation for MVP: add a `registration_blocked` boolean or `status` enum to the seller profile model; no dedicated blacklist table needed at MVP scale | Medium | Phase 4 (Architecture) — confirm data model approach; Phase 7 (Implementation) — implement registration block |
| A-03-011 | EMI availability at checkout (Gap G-002 resolution) requires no ShopNest-side gating logic — Razorpay's checkout widget handles EMI eligibility determination at the bank level | Razorpay's standard checkout handles EMI eligibility natively; ShopNest only needs to pass the order amount and not explicitly exclude EMI from the payment_methods list sent to the Razorpay Orders API | High | Phase 7 (Implementation) — confirm Razorpay Orders API `payment_capture` and `method` parameters permit EMI passthrough |
| A-03-012 | 60-event taxonomy (38 P1 + 18 P2 + 4 P3) across 12 namespaces is comprehensive for MVP analytics coverage without being excessive for a 3-developer team to instrument | 60 events is standard for a platform of this scope at MVP (Mixpanel's typical e-commerce starter taxonomy is 40–80 events); P1/P2/P3 prioritization allows the team to ship P1 events first and add P2/P3 iteratively | Medium | Phase 9 (Testing) — TC-ANLX-001 through TC-ANLX-010 validate all P1 events |
| A-03-013 | Admin dashboards built in Phase 13 (Monitoring) rather than Phase 7 (Implementation) — analytics instrumentation (events + endpoint) is Phase 7 work; dashboard UI is Phase 13 work | Separating instrumentation from visualization is correct SDLC phasing; dashboards require data to be flowing in production before they can be meaningfully configured; Phase 13 is the correct phase for observability and monitoring setup | High | Phase 7 (Implementation) — instrumentation complete; Phase 13 (Monitoring) — dashboard construction |

---

## Conflicts Resolved This Phase

| Conflict ID | Description | Resolution | Resolution Authority |
|-------------|-------------|-----------|---------------------|
| C-001 | "Sellers keep 100% of transaction revenue" (Phase 1 value prop) vs. "net of Razorpay processing fee" (Phase 2 BR-005) | Clarified in PRD §1: ShopNest charges zero *platform* commission; Razorpay gateway fee (~2%+GST) applies universally as it is charged by Razorpay to ShopNest, not by ShopNest to sellers. ShopNest absorbs this as an operating cost of the zero-commission model. PRD §6 feature specifications use "net of Razorpay gateway fee" as the canonical description. | Phase 3 PRD synthesis (Tier 3 resolution — no Tier 1 input required; both source documents approved) |
| C-002 | Inconsistent terminology: "Pending Review" vs. "Pending Admin Approval" across USER-STORIES.md | Canonical term = **"Pending Review"** for both seller accounts and product listings throughout PRD and all downstream artifacts. Phase 4+ agents must use this term. | Phase 3 PRD synthesis |

---

## Gaps Resolved This Phase

| Gap ID | Description | Resolution | Resolution Authority |
|--------|-------------|-----------|---------------------|
| G-001 | Rejected seller re-registration behaviour not specified in Phase 2 | Rejected seller's email is blocked from re-registration. Seller must contact ShopNest support to appeal. Implemented as a `registration_blocked` flag or status check on the seller profile model (see A-03-010). Added to PRD §13 as OQ-001: confirm UX for rejection reason display. | Phase 3 PRD synthesis |
| G-002 | EMI mentioned in Phase 1 value propositions but omitted from Phase 2 FR-CHECKOUT-004 payment method list | EMI included as an available checkout option via Razorpay's native checkout widget. No ShopNest-side gating required. FR-CHECKOUT-004 should be updated in a Phase 2 errata or carried into Phase 4 data model with a note. | Phase 3 PRD synthesis |

---

## Open Flags (Tier 2 — Suggestions Made This Phase)

| Flag ID | Suggestion Made | Accepted By | Status |
|---------|----------------|-------------|--------|
| F-03-001 | Analytics events purge policy: 2 years for behavioral events, 7 years for financial-adjacent events | Tier 3 inference (A-03-004) — not presented to user as a Tier 2 suggestion because the 7-year retention rule is already defined in BR-011 and the 2-year behavioral cutoff is a standard operational practice | N/A — handled as Tier 3 |
| F-03-002 | Feature flags implemented as env vars, not LaunchDarkly or similar | Tier 3 inference (A-03-008) — consistent with 3-developer team budget constraint | N/A — handled as Tier 3 |

---

## Phase 1 and Phase 2 Assumptions Carried Forward or Resolved

| Prior ID | Original Assumption | Phase 3 Status | Notes |
|----------|--------------------|--------------|----|
| A-01-016 | ₹1,999/month pricing acceptable to sellers | Carried forward — unvalidated | PRD OQ-003 tracks this; beta seller pricing interviews required pre-launch |
| A-02-001 | Low-stock threshold = 5 units | Carried forward | Validation: Phase 9 TC-010 |
| A-02-004 | Razorpay Payouts API with NEFT/IMPS | Carried forward | Validation: Phase 7 sandbox testing |
| A-02-006 | Monday 09:00 IST payout execution time | Carried forward — medium confidence | PRD OQ-002 flags this as stakeholder-confirmable |
| A-02-008 | Seller ledger credits order amount net of Razorpay fee (~2%+GST) | **Resolved as C-001** — now canonical in PRD §6 | Carry forward to Phase 4 data model: payout ledger entry must store gross_amount_inr and gateway_fee_inr |

---

## Resolution Log

| ID | Original Assumption | Resolution | Resolved By | Date |
|----|--------------------|-----------|-----------:|------|
| A-03-001 | Internal-only analytics (no third-party SDK) | Accepted as MVP approach | Phase 3 agent (Tier 3) | 2026-05-05 |
| A-03-007 | Prices in paise (integer) in analytics events | Accepted as engineering standard | Phase 3 agent (Tier 3) | 2026-05-05 |
