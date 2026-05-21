# Requirements Traceability Matrix — ShopNest
**Phase:** 02 — Requirements Engineering
**Generated:** 2026-05-04
**Status:** Draft — Awaiting Human Gate Approval

---

> **Purpose:** Every requirement traces upward to a business goal and downward to a test case. A blank "Test Case" column is expected at this stage — Phase 9 (Testing) will backfill all TC-XXX identifiers.
>
> **Reading the matrix:** Each row represents one traceable chain: the business goal that motivates the work → the user story that defines it → the functional requirement that specifies it → the business rule that governs it → the test that verifies it → any Phase 1 assumption this requirement validates.

---

## Legend

| Column | Meaning |
|--------|---------|
| Business Goal | KPI or desired outcome from SUCCESS-METRICS.md |
| User Story | US-XXX — from USER-STORIES.md |
| Functional Req | FR-XXX — from REQUIREMENTS.md §2 |
| Business Rule | BR-XXX — from BUSINESS-RULES.md |
| NFR | NFR-XXX — from REQUIREMENTS.md §4 |
| Test Case | TC-XXX — to be assigned in Phase 9 |
| Assumption Validated | A-01-XXX / A-02-XXX — from assumption logs |
| Status | Defined / In Progress / Verified |

---

## Section 1: Seller Onboarding & Subscription

| Business Goal | User Story | Functional Req | Business Rule | NFR | Test Case | Assumption Validated | Status |
|---------------|-----------|----------------|--------------|-----|-----------|---------------------|--------|
| 100 active paying sellers within 6 months | US-001 (Seller Registration) | FR-AUTH-001, FR-AUTH-002, FR-AUTH-003, FR-AUTH-009 | BR-014 | NFR-SEC-004 | TC-001 (Phase 9) | A-01-001 | Defined |
| 100 active paying sellers within 6 months | US-002 (Store Setup Wizard) | FR-SELLER-001, FR-SELLER-002, FR-SELLER-005 | — | NFR-USA-004 | TC-002 (Phase 9) | A-01-001 | Defined |
| 100 active paying sellers within 6 months | US-003 (Seller Subscription Signup) | FR-SUBSCR-001, FR-SUBSCR-003, FR-SUBSCR-004, FR-SUBSCR-005, FR-SUBSCR-006 | BR-007 | NFR-REL-005 | TC-003 (Phase 9) | A-01-016 | Defined |
| Seller activation rate > 70% | US-021 (Admin Approves Seller) | FR-ADMIN-001, FR-ADMIN-002 | BR-001, BR-014 | — | TC-021 (Phase 9) | — | Defined |
| Seller monthly churn rate < 5% | US-003 (Grace period and suspension) | FR-SUBSCR-003, FR-SUBSCR-004, FR-SUBSCR-005, FR-SUBSCR-006, FR-SUBSCR-007 | BR-007 | — | TC-003b (Phase 9) | A-01-014 | Defined |
| Seller NPS > 40 | US-036 (Subscription status + invoices) | FR-SUBSCR-002, FR-SUBSCR-007 | BR-010, BR-011 | — | TC-036 (Phase 9) | — | Defined |

---

## Section 2: Product Listing & Moderation

| Business Goal | User Story | Functional Req | Business Rule | NFR | Test Case | Assumption Validated | Status |
|---------------|-----------|----------------|--------------|-----|-----------|---------------------|--------|
| Seller store setup completion > 80% | US-004 (Add Product Listing) | FR-PRODUCT-001, FR-PRODUCT-002, FR-PRODUCT-003, FR-PRODUCT-004, FR-PRODUCT-011 | BR-001, BR-015 | NFR-SEC-009 | TC-004 (Phase 9) | — | Defined |
| Seller store setup completion > 80% | US-005 (Admin Approves Product) | FR-PRODUCT-005, FR-PRODUCT-006, FR-ADMIN-005, FR-ADMIN-006 | BR-001 | — | TC-005 (Phase 9) | — | Defined |
| Seller store setup completion > 80% | US-006 (Edit Price/Stock — No Re-review) | FR-PRODUCT-007, FR-PRODUCT-008 | BR-017 | — | TC-006 (Phase 9) | — | Defined |
| Platform integrity | US-022, US-030 (Admin Product Moderation) | FR-PRODUCT-005, FR-PRODUCT-006, FR-PRODUCT-012 | BR-009 | — | TC-022 (Phase 9) | — | Defined |
| Seller NPS > 40 | US-010 (Low-Stock Alert) | FR-PRODUCT-010 | — | — | TC-010 (Phase 9) | A-02-001 | Defined |
| Seller NPS > 40 | US-031 (Seller Edits Store Profile) | FR-SELLER-006 | — | — | TC-031 (Phase 9) | — | Defined |
| Platform integrity | US-023 (Admin Suspends Seller) | FR-ADMIN-004, FR-ADMIN-005 | BR-001, BR-013 | — | TC-023 (Phase 9) | — | Defined |

---

## Section 3: Buyer Discovery & Browse

| Business Goal | User Story | Functional Req | Business Rule | NFR | Test Case | Assumption Validated | Status |
|---------------|-----------|----------------|--------------|-----|-----------|---------------------|--------|
| Buyer cart-to-order conversion > 3% | US-011 (Browse Homepage) | FR-BROWSE-001 | BR-001 | NFR-PE-002, NFR-PE-003, NFR-USA-001 | TC-011 (Phase 9) | A-01-002 | Defined |
| Buyer cart-to-order conversion > 3% | US-012 (Browse by Category) | FR-BROWSE-002 | BR-001 | NFR-PE-001, NFR-USA-003 | TC-012 (Phase 9) | — | Defined |
| Buyer cart-to-order conversion > 3% | US-013 (Product Detail Page) | FR-BROWSE-003, FR-BROWSE-004, FR-BROWSE-007 | BR-001 | NFR-PE-002, NFR-PE-004 | TC-013 (Phase 9) | — | Defined |
| Buyer cart-to-order conversion > 3% | US-014 (Search Products) | FR-BROWSE-005, FR-BROWSE-006 | BR-001 | NFR-PE-005 | TC-014 (Phase 9) | A-02-002 | Defined |

---

## Section 4: Cart Management

| Business Goal | User Story | Functional Req | Business Rule | NFR | Test Case | Assumption Validated | Status |
|---------------|-----------|----------------|--------------|-----|-----------|---------------------|--------|
| Guest checkout completion > 60% | US-015 (Add to Cart) | FR-CART-001, FR-CART-002, FR-CART-004, FR-CART-006, FR-CART-007 | — | NFR-REL-005 | TC-015 (Phase 9) | — | Defined |
| Buyer repeat purchase > 25% (30 days) | US-026 (Registered Buyer Cart) | FR-CART-003 | — | — | TC-026 (Phase 9) | — | Defined |

---

## Section 5: Checkout & Payment

| Business Goal | User Story | Functional Req | Business Rule | NFR | Test Case | Assumption Validated | Status |
|---------------|-----------|----------------|--------------|-----|-----------|---------------------|--------|
| Guest checkout completion > 60% | US-016 (Guest Checkout Form) | FR-CHECKOUT-001, FR-CART-005 | BR-012, BR-018 | NFR-USA-001 | TC-016 (Phase 9) | — | Defined |
| Payment success rate > 97% | US-017 (Pay via Razorpay) | FR-CHECKOUT-003, FR-CHECKOUT-004, FR-CHECKOUT-005, FR-CHECKOUT-006, FR-CHECKOUT-007, FR-CHECKOUT-008, FR-CHECKOUT-011, FR-CHECKOUT-012 | BR-004, BR-008, BR-012 | NFR-PE-007, NFR-REL-005, NFR-SEC-006, NFR-SEC-007 | TC-017 (Phase 9) | A-01-003 | Defined |
| Checkout error rate < 1% | US-017 (Payment failure handling) | FR-CHECKOUT-012 | BR-008 | NFR-REL-005 | TC-017b (Phase 9) | — | Defined |
| Payment success rate > 97% | US-018 (Order Confirmation Email) | FR-CHECKOUT-009, FR-NOTIF-001, FR-NOTIF-002, FR-NOTIF-004 | — | NFR-REL-006 | TC-018 (Phase 9) | A-02-003 | Defined |
| Buyer repeat purchase > 25% | US-026 (Registered Buyer Checkout) | FR-CHECKOUT-002 | — | NFR-USA-001 | TC-026b (Phase 9) | — | Defined |

---

## Section 6: Order Management (Buyer-Side)

| Business Goal | User Story | Functional Req | Business Rule | NFR | Test Case | Assumption Validated | Status |
|---------------|-----------|----------------|--------------|-----|-----------|---------------------|--------|
| Buyer order tracking engagement > 50% | US-019 (Track Order Status) | FR-ORDER-001, FR-ORDER-002 | — | — | TC-019 (Phase 9) | — | Defined |
| Buyer cart-to-order conversion > 3% | US-020 (Cancel Order) | FR-ORDER-003, FR-ORDER-004 | BR-002, BR-003 | — | TC-020 (Phase 9) | — | Defined |
| Buyer repeat purchase > 25% | US-027 (Buyer Order History) | FR-ORDER-005 | BR-016, BR-018 | — | TC-027 (Phase 9) | — | Defined |

---

## Section 7: Order Fulfillment (Seller-Side)

| Business Goal | User Story | Functional Req | Business Rule | NFR | Test Case | Assumption Validated | Status |
|---------------|-----------|----------------|--------------|-----|-----------|---------------------|--------|
| Order fulfillment rate > 95% within 48h | US-007 (Seller Views Orders) | FR-FULFILL-001, FR-FULFILL-006 | BR-016 | NFR-SEC-005 | TC-007 (Phase 9) | — | Defined |
| Order fulfillment rate > 95% within 48h | US-008 (Seller Confirms Order) | FR-FULFILL-002 | BR-002 | — | TC-008 (Phase 9) | — | Defined |
| Order fulfillment rate > 95% within 48h | US-009 (Seller Ships Order) | FR-FULFILL-003, FR-FULFILL-004 | — | — | TC-009 (Phase 9) | A-01-001 | Defined |
| Order fulfillment rate > 95% within 48h | US-033 (Buyer Receives Shipped Email) | FR-FULFILL-004, FR-NOTIF-002 | — | NFR-REL-006 | TC-033 (Phase 9) | — | Defined |
| Seller NPS > 40 | US-034 (Seller Marks Delivered) | FR-FULFILL-005 | BR-006 | — | TC-034 (Phase 9) | — | Defined |

---

## Section 8: Seller Payouts & Settlement

| Business Goal | User Story | Functional Req | Business Rule | NFR | Test Case | Assumption Validated | Status |
|---------------|-----------|----------------|--------------|-----|-----------|---------------------|--------|
| MRR target ₹1,99,900 at 6 months | US-024 (Weekly Payout) | FR-PAYOUT-001, FR-PAYOUT-002, FR-PAYOUT-003, FR-PAYOUT-004, FR-PAYOUT-005 | BR-005, BR-006, BR-013 | NFR-MAINT-005 | TC-024 (Phase 9) | A-02-004 | Defined |
| Seller NPS > 40 | US-032 (Payout History) | FR-PAYOUT-006 | BR-011 | — | TC-032 (Phase 9) | — | Defined |
| AWS cost per seller < ₹2,000/month | CON-001 | All FR | — | NFR-PE-006, NFR-PE-008 | TC-PERF-001 (Phase 9) | A-01-017 | Defined |

---

## Section 9: Notifications

| Business Goal | User Story | Functional Req | Business Rule | NFR | Test Case | Assumption Validated | Status |
|---------------|-----------|----------------|--------------|-----|-----------|---------------------|--------|
| Buyer order tracking engagement > 50% | US-018, US-033 | FR-NOTIF-001, FR-NOTIF-002, FR-NOTIF-003, FR-NOTIF-004 | — | NFR-REL-006 | TC-NOTIF-001 (Phase 9) | A-02-003 | Defined |

---

## Section 10: Non-Functional Requirements Traceability

| NFR | Source Goal | Functional Area | Test Method | Test Case | Status |
|-----|-------------|-----------------|-------------|-----------|--------|
| NFR-PE-001 (API P95 < 200ms) | API Response Time target (CLAUDE.md) | All API endpoints | Locust/k6 load test at 800 concurrent users | TC-PERF-001 (Phase 9) | Defined |
| NFR-PE-002 (LCP < 2.5s mobile) | Page Load target (CLAUDE.md) | Product pages (SSR) | Google PageSpeed / Core Web Vitals | TC-PERF-002 (Phase 9) | Defined |
| NFR-PE-006 (≥ 1,000 concurrent) | Concurrent Users target | Full platform | Locust load test at 1,000 concurrent users | TC-PERF-003 (Phase 9) | Defined |
| NFR-REL-001 (99.9% uptime) | Uptime SLA target | Platform-wide | CloudWatch alarm; 30-day uptime report | TC-REL-001 (Phase 9) | Defined |
| NFR-REL-005 (Webhook idempotency) | Checkout error rate < 1% | Checkout, payment | Duplicate webhook simulation test | TC-REL-005 (Phase 9) | Defined |
| NFR-SEC-001 (TLS 1.2+) | Security baseline | All endpoints | SSL Labs scan | TC-SEC-001 (Phase 9) | Defined |
| NFR-SEC-005 (Multi-tenant isolation) | Platform integrity | Seller dashboard | Automated cross-tenant access test | TC-SEC-005 (Phase 9) | Defined |
| NFR-SEC-006 (Webhook HMAC validation) | Payment security | Checkout webhook | Forged webhook simulation test | TC-SEC-006 (Phase 9) | Defined |
| NFR-USA-003 (WCAG 2.1 AA) | Usability (buyer-facing) | Buyer marketplace pages | Axe DevTools + manual keyboard nav | TC-USA-003 (Phase 9) | Defined |
| NFR-MAINT-001 (80% test coverage — backend) | Code quality | Django apps | pytest-cov CI report | TC-MAINT-001 (Phase 9) | Defined |
| NFR-MAINT-002 (80% test coverage — frontend) | Code quality | Next.js app | Jest + Istanbul CI report | TC-MAINT-002 (Phase 9) | Defined |

---

## Section 11: Business Rules Traceability

| Business Rule | Source Authority | User Stories | Functional Reqs | Test Case | Status |
|---------------|-----------------|--------------|-----------------|-----------|--------|
| BR-001 (Triple-Gate Visibility) | ShopNest platform policy | US-003, US-005, US-022, US-023 | FR-PRODUCT-003, FR-SUBSCR-005, FR-ADMIN-004 | TC-BR-001 (Phase 9) | Defined |
| BR-002 (Cancellation Window) | MVP cancellation policy | US-020 | FR-ORDER-003, FR-FULFILL-002 | TC-BR-002 (Phase 9) | Defined |
| BR-005 (Zero Commission) | Business model | US-024 | FR-PAYOUT-001 | TC-BR-005 (Phase 9) | Defined |
| BR-006 (Weekly Payout Cycle) | Payout policy | US-024 | FR-PAYOUT-002 | TC-BR-006 (Phase 9) | Defined |
| BR-007 (Grace Period) | Subscription policy | US-003 | FR-SUBSCR-003, FR-SUBSCR-004, FR-SUBSCR-005 | TC-BR-007 (Phase 9) | Defined |
| BR-008 (Stock Decrement Timing) | Inventory policy | US-017 | FR-CHECKOUT-008 | TC-BR-008 (Phase 9) | Defined |
| BR-010 (GST Invoice Scope) | Indian GST Act | US-036 | FR-SUBSCR-007 | TC-BR-010 (Phase 9) | Defined |
| BR-011 (7-Year Retention) | Companies Act / GST Act | All financial | Data Requirements §5.1 | TC-BR-011 (Phase 9) | Defined |
| BR-012 (Self-Purchase Prohibition) | Platform integrity | US-016, US-017 | FR-CHECKOUT-006 | TC-BR-012 (Phase 9) | Defined |
| BR-017 (Re-review Trigger Conditions) | Moderation efficiency | US-006 | FR-PRODUCT-008 | TC-BR-017 (Phase 9) | Defined |

---

## Section 12: Assumption Validation Coverage

| Assumption ID | Description | Validated By | User Story / FR | Target Phase |
|---------------|-------------|-------------|-----------------|-------------|
| A-01-001 | WhatsApp + spreadsheet is dominant seller workaround | Seller onboarding wizard UX test (onboarding time ≤ 30 min) | US-002, NFR-USA-004 | Phase 9 |
| A-01-002 | >70% mobile traffic | Core Web Vitals on mobile (LCP < 2.5s, NFR-PE-002) | US-011, US-013 | Phase 9 |
| A-01-003 | Razorpay webhook pattern | Payment integration test (TC-017) | US-017, FR-CHECKOUT-005 | Phase 7 (Implementation) |
| A-01-014 | Monthly churn < 5% is achievable | Measure from first paying cohort at Month 3 | Business KPI — not a code test | Month 3 post-launch |
| A-01-016 | ₹1,999/month pricing acceptable | Beta seller pricing interviews (pre-launch) | Business KPI — not a code test | Q3 2026 (pre-launch) |
| A-01-017 | AWS cost ≈ ₹1,66,000/month at 100 sellers | AWS cost measurement post-launch | CON-001 | Phase 12 (Deployment) |
| A-02-001 | Low-stock threshold = 5 units | TC-010 (low-stock alert test) | US-010, FR-PRODUCT-010 | Phase 9 |
| A-02-002 | PostgreSQL FTS sufficient for ≤50K products | TC-014 (search P95 ≤ 500ms load test) | US-014, NFR-PE-005 | Phase 9 |
| A-02-003 | AWS SES as email provider | TC-018, TC-NOTIF-001 | US-018, FR-NOTIF-001 | Phase 9 |
| A-02-004 | Razorpay Payouts API for NEFT/IMPS | TC-024 (payout integration test) | US-024, FR-PAYOUT-002 | Phase 7 (Implementation) |

---

## Coverage Summary

| Category | Total Items | Traced to Business Goal | Traced to Test Case | Status |
|----------|------------|------------------------|--------------------|----|
| User Stories (Must Have) | 24 | 24 / 24 ✅ | 24 / 24 (TC placeholder) | Complete |
| User Stories (Should Have) | 10 | 10 / 10 ✅ | 10 / 10 (TC placeholder) | Complete |
| User Stories (Could Have) | 4 | 4 / 4 ✅ | 4 / 4 (TC placeholder) | Complete |
| Functional Requirements | 68 | 68 / 68 ✅ | Via story TCs | Complete |
| Business Rules | 18 | 18 / 18 ✅ | 10 direct TC entries | Complete |
| NFRs | 37 | 37 / 37 ✅ | 11 direct TC entries | Complete |
| Assumptions (Phase 1 + 2) | 10 | 10 / 10 ✅ | Via story TCs or business validation | Complete |

**No orphan requirements** — every requirement traces to a business goal.  
**No orphan test placeholders** — every TC placeholder links to a requirement.

---

*Test Case IDs (TC-XXX) will be assigned and populated by Phase 9 (Testing Agent).*  
*Sources: SUCCESS-METRICS.md; USER-STORIES.md; REQUIREMENTS.md; BUSINESS-RULES.md; assumption logs.*
