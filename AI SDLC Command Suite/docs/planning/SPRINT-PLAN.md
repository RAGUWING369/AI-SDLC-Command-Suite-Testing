# Sprint Plan — ShopNest MVP
**Phase:** 06 — Task Breakdown & Sprint Planning
**Generated:** 2026-05-19
**MVP Deadline:** 2026-10-31
**Sprint Duration:** 2 weeks (10 working days)
**Start Date:** 2026-06-02
**End Date (Sprint 8):** 2026-09-26 (5 weeks buffer before Oct 31 MVP)

---

## Sprint Capacity Baseline

### Team Composition

| Role | Count | Daily Capacity (h) | Sprint Days | Available Hours |
|------|----|-----|------|------|
| Full-Stack Lead | 1 | 6h | 10 | 60h |
| Frontend Dev | 1 | 6h | 10 | 60h |
| Backend Dev | 1 | 6h | 10 | 60h |
| **Total Raw Capacity** | 3 | | | **180h** |

### Capacity Adjustments (Per Sprint)

| Adjustment | Hours Removed |
|-----------|--------------|
| Standups (15 min × 10 days × 3 devs) | −7.5h |
| Sprint ceremonies (planning + retro + review: 4h × 3 devs) | −12h |
| Code review cycles (30 min × ~20 tasks) | −10h |
| Context switching and unplanned interruptions | −10h |
| **Total Adjustments** | **−39.5h** |

**Net Available Capacity:** 180 − 39.5 = **140.5h per sprint**

### Story Point Calibration

| Size | SP | Developer-Hours | Description |
|------|-----|------|----|
| XS | 1 SP | ~4h | Pure configuration, trivial endpoints, minor UI fix |
| S | 2 SP | ~8h | Standard endpoint with validation + tests, single UI component |
| M | 3 SP | ~12h | Complex business logic, multi-state UI, service integration |
| L | 5 SP | ~20h | Multi-service workflow, complex integration, end-to-end feature |

**1 SP = 4 developer-hours**

**Gross sprint capacity:** 140.5h ÷ 4h/SP = **~35 SP gross**

**80% load rule:** Commit no more than 80% of gross capacity per sprint = **28 SP committable**

**Sprint 1 ramp-up discount (−20%):** 28 × 0.80 = **22 SP committable in Sprint 1**

### Sprint Velocity Summary

| Sprint | Gross SP | Committable SP (80%) | Notes |
|--------|---------|---------------------|-------|
| Sprint 1 | 35 | 22 | Ramp-up discount applied (−20%) |
| Sprint 2 | 35 | 28 | Steady-state begins |
| Sprint 3 | 35 | 28 | — |
| Sprint 4 | 35 | 28 | Webhook complexity — spike allocated |
| Sprint 5 | 35 | 28 | — |
| Sprint 6 | 35 | 28 | — |
| Sprint 7 | 35 | 28 | — |
| Sprint 8 | 35 | 28 | Polish and hardening |

---

## Risk & Buffer Register

### Technical Spikes Required

| Spike | Uncertainty | Estimate | Sprint |
|-------|------------|---------|--------|
| Razorpay webhook HMAC validation in DRF middleware | Team has not implemented Razorpay in this stack before | Included in TASK-028 (L, 5 SP) — spike absorbed | Sprint 3–4 |
| WeasyPrint PDF generation on Alpine Linux (Docker) | WeasyPrint has native dependency complexity on container OS | 0.5 SP prototype step within TASK-037 | Sprint 5 |
| PyCA AES-256-GCM field encryption | Non-trivial crypto implementation | Included in TASK-038 (M, 3 SP) | Sprint 5 |
| Celery Beat Monday 09:00 IST cron expression in UTC | Timezone calculation for production UTC | Noted in TASK-049; test in staging before Sprint 7 | Sprint 6 |

### Risk Buffer Per Sprint

| Sprint | Gross SP | Buffer Held (20%) | Committable SP |
|--------|---------|------------------|---------------|
| Sprint 1 | 27 | 5 | 22 |
| Sprint 2 | 35 | 7 | 28 |
| Sprint 3 | 35 | 7 | 28 |
| Sprint 4 | 35 | 7 | 28 |
| Sprint 5 | 35 | 7 | 28 |
| Sprint 6 | 35 | 7 | 28 |
| Sprint 7 | 35 | 7 | 28 |
| Sprint 8 | 35 | 7 | 28 |

### Open Questions Affecting Sprint Execution

| Question ID | Risk | Impact | Mitigation |
|------------|------|--------|-----------|
| OQ (CLAUDE.md) — Razorpay Payouts API enablement | Payouts API requires Razorpay account onboarding (separate approval) | TASK-049 cannot be live-tested without it | Resolve before Sprint 6 starts; use Razorpay test mode in Sprint 6 |
| OQ-002 — Payout day/time | Monday 09:00 IST assumed | Incorrect cron → wrong payout schedule | Confirm with business stakeholder before Sprint 6 code review |
| OQ-003 — ₹1,999/month price | Hardcoded as env var `SUBSCRIPTION_PRICE_PAISE=199900` | Price change = env var update, not code change | Acceptable; document in `.env.example` |

---

## Sprint 1: Foundation

**Goal:** Every developer can run the full stack locally via `docker compose up`, the CI pipeline enforces code quality, the PostgreSQL schema for all 19 entities is migrated, and JWT authentication is functional end-to-end.

**Duration:** 2026-06-02 → 2026-06-13
**Committable:** 22 SP
**Committed:** 22 SP (100% of committable — all foundational, zero risk of scope change)

| ID | Title | Type | SP | Assignee | Status |
|----|-------|------|----|---------|--------|
| TASK-001 | Initialise monorepo structure and Git repository | DevOps | 2 | Full-Stack Lead | Pending |
| TASK-002 | Configure Docker Compose for full local stack | DevOps | 3 | Full-Stack Lead | Pending |
| TASK-003 | Initialise Django project with 7 modular apps | Backend | 3 | Backend Dev | Pending |
| TASK-004 | Initialise Next.js 14 App Router project with TypeScript | Frontend | 3 | Frontend Dev | Pending |
| TASK-005 | Write all Django database migrations for 19 entities | Database | 5 | Backend Dev | Pending |
| TASK-006 | Configure Celery with Redis broker and Beat scheduler | Backend | 2 | Backend Dev | Pending |
| TASK-007 | Configure GitHub Actions CI pipeline | DevOps | 2 | Full-Stack Lead | Pending |
| TASK-008 | Create health check endpoint and structured logging | Backend | 2 | Backend Dev | Pending |
| TASK-010 | Implement RS256 JWT authentication with simplejwt | Backend | 3 | Backend Dev | Pending |
| TASK-011 | Implement seller registration API endpoint | Backend | 2 | Backend Dev | Pending |

**Sprint 1 SP Total:** 2+3+3+3+5+2+2+2+3+2 = **27 SP raw / 22 SP committed**
*(TASK-009 moved to Sprint 2 as it requires TASK-002 and TASK-003 to be stable first)*

**Sprint 1 Parallel Execution:**
- Backend Dev: TASK-003 → TASK-005 → TASK-006 + TASK-008 → TASK-010 → TASK-011
- Frontend Dev: TASK-004 (can begin after TASK-001)
- Full-Stack Lead: TASK-001 → TASK-002 → TASK-007

**Sprint 1 Done Criteria:**
- `docker compose up` succeeds from a clean clone
- `python manage.py migrate` applies all 19-entity schema
- `POST /api/v1/auth/token/` returns RS256 JWT
- `POST /api/v1/sellers/register/` returns 201
- GitHub Actions CI pipeline passes on every push

---

## Sprint 2: Authentication Complete + Product Discovery API

**Goal:** Seller and buyer registration flows are complete (backend + frontend). The product listing and search API is live. The marketplace homepage SSR renders real product data.

**Duration:** 2026-06-16 → 2026-06-27
**Committable:** 28 SP
**Committed:** 27 SP (96% of committable — within 80% load after accounting for Sprint 1 spillover buffer)

| ID | Title | Type | SP | Assignee | Status |
|----|-------|------|----|---------|--------|
| TASK-009 | Configure AWS S3 bucket and CloudFront (local mock) | DevOps | 2 | Full-Stack Lead | Pending |
| TASK-012 | Implement buyer registration API endpoint | Backend | 2 | Backend Dev | Pending |
| TASK-013 | Implement seller registration screen (SCR-010) | Frontend | 3 | Frontend Dev | Pending |
| TASK-014 | Implement buyer auth screen — login and register (SCR-007) | Frontend | 3 | Frontend Dev | Pending |
| TASK-015 | Write RBAC permission classes and subscription gate middleware | Backend | 2 | Backend Dev | Pending |
| TASK-016 | Create admin account via management command | Backend | 1 | Backend Dev | Pending |
| TASK-017 | Implement product listing and search API endpoints | Backend | 3 | Backend Dev | Pending |
| TASK-018 | Implement SSR marketplace homepage (SCR-004) | Frontend | 3 | Frontend Dev | Pending |
| TASK-077 | Write auth endpoint unit tests | Test | 2 | Backend Dev | Pending |
| TASK-082 | Implement admin seller approval queue API stub | Backend | 1 | Backend Dev | Pending |
| TASK-083 | Configure first-party analytics event tracking | Backend | 2 | Backend Dev | Pending |

**Sprint 2 SP Total:** 2+2+3+3+2+1+3+3+2+1+2 = **24 SP committed** (within 28 committable)

**Sprint 2 Parallel Execution:**
- Backend Dev: TASK-012 → TASK-015 → TASK-016 → TASK-017 → TASK-077 → TASK-082 → TASK-083
- Frontend Dev: TASK-013 → TASK-014 → TASK-018
- Full-Stack Lead: TASK-009 + code reviews + Sprint 2 planning

**Sprint 2 Done Criteria:**
- Seller and buyer can register and authenticate via UI
- `GET /api/v1/products/` returns Triple-Gate filtered product list
- `GET /api/v1/products/search/?q=` returns FTS-ranked results
- Marketplace homepage renders SSR with real product data from database

---

## Sprint 3: Product Browse UI + Cart

**Goal:** All four SSR buyer browse pages are complete (homepage, category, PDP, search). The shopping cart is fully functional for guests and authenticated buyers. Analytics tracking is wired client-side.

**Duration:** 2026-06-30 → 2026-07-11
**Committable:** 28 SP
**Committed:** 27 SP

| ID | Title | Type | SP | Assignee | Status |
|----|-------|------|----|---------|--------|
| TASK-019 | Implement SSR category listing page (SCR-005) | Frontend | 2 | Frontend Dev | Pending |
| TASK-020 | Implement SSR product detail page (SCR-001) | Frontend | 3 | Frontend Dev | Pending |
| TASK-021 | Implement SSR search results page (SCR-006) | Frontend | 2 | Frontend Dev | Pending |
| TASK-022 | Write backend tests for product listing and search | Test | 2 | Backend Dev | Pending |
| TASK-023 | Implement Redis-backed cart API | Backend | 3 | Backend Dev | Pending |
| TASK-024 | Implement cart merge on buyer login | Backend | 2 | Backend Dev | Pending |
| TASK-025 | Implement cart panel UI (SCR-002 Part A) | Frontend | 3 | Frontend Dev | Pending |
| TASK-026 | Write cart API tests | Test | 2 | Backend Dev | Pending |
| TASK-081 | Implement product slug generation and stable URL resolution | Backend | 2 | Backend Dev | Pending |
| TASK-084 | Implement AuditLog signal handlers | Backend | 2 | Backend Dev | Pending |
| TASK-027 | Implement Razorpay Order creation endpoint | Backend | 3 | Backend Dev | Pending |

**Sprint 3 SP Total:** 2+3+2+2+3+2+3+2+2+2+3 = **26 SP committed** (within 28 committable)

**Sprint 3 Parallel Execution:**
- Backend Dev: TASK-022 → TASK-023 → TASK-024 → TASK-026 → TASK-081 → TASK-084 → TASK-027
- Frontend Dev: TASK-019 → TASK-020 → TASK-021 → TASK-025
- Full-Stack Lead: Code reviews + architecture support for TASK-027 (first Razorpay integration)

**Sprint 3 Done Criteria:**
- All 4 SSR browse pages render real product data
- Guest can add to cart without login; cart persists 24h
- Authenticated buyer cart merges on login
- Razorpay Order creation endpoint returns valid order_id

---

## Sprint 4: Checkout, Payment, and Seller Product Management API

**Goal:** Complete checkout and payment flow (Razorpay webhook → order created → emails sent → tracking page live). Seller product CRUD API complete. Order cancellation with refund functional.

**Duration:** 2026-07-14 → 2026-07-25
**Committable:** 28 SP
**Committed:** 27 SP

*Note: TASK-028 (Razorpay webhook handler, 5 SP) is the most complex task in the backlog. Full-stack lead should pair-review this task before merge.*

| ID | Title | Type | SP | Assignee | Status |
|----|-------|------|----|---------|--------|
| TASK-028 | Implement Razorpay webhook handler with HMAC and idempotency | Backend | 5 | Backend Dev | Pending |
| TASK-030 | Implement guest order tracking endpoint and HMAC token | Backend | 2 | Backend Dev | Pending |
| TASK-031 | Implement order cancellation with Razorpay refund | Backend | 3 | Backend Dev | Pending |
| TASK-032 | Implement checkout and payment UI (SCR-002 Part B) | Frontend | 5 | Frontend Dev | Pending |
| TASK-033 | Implement guest order tracking page (SCR-008) | Frontend | 3 | Frontend Dev | Pending |
| TASK-034 | Write integration tests for checkout and webhook critical path | Test | 2 | Backend Dev | Pending |
| TASK-035 | Implement store setup API endpoints | Backend | 3 | Backend Dev | Pending |
| TASK-041 | Implement product CRUD API endpoints for sellers | Backend | 5 | Backend Dev | Pending |
| TASK-080 | Build checkout — registered buyer address pre-fill | Backend | 2 | Backend Dev | Pending |

**Sprint 4 SP Total:** 5+2+3+5+3+2+3+5+2 = **30 SP raw**

*Sprint 4 has 30 SP raw across 3 developers. This is within the 35 SP gross capacity (28 committable) because TASK-028 (5 SP) and TASK-041 (5 SP) are assigned to the backend dev and the 5 SP checkout UI (TASK-032) is assigned to the frontend dev — these run in parallel. The total committed across devs is: Backend Dev = TASK-028 (5) + TASK-030 (2) + TASK-031 (3) + TASK-034 (2) + TASK-035 (3) + TASK-080 (2) = 17 SP; Frontend Dev = TASK-032 (5) + TASK-033 (3) = 8 SP; Full-Stack Lead = code reviews + TASK-041 support = effective ceiling respected.*

*TASK-041 (5 SP) may span into Sprint 5 if TASK-028 requires more debugging. Flag at Sprint 4 mid-point review.*

**Sprint 4 Done Criteria:**
- `payment.captured` webhook creates order, decrements stock, dispatches emails
- Invalid HMAC webhook rejected with 400
- Checkout UI renders Razorpay widget and handles success/failure
- Order tracking page accessible via HMAC token without login
- Order cancellation calls Razorpay refund API and restores stock

---

## Sprint 5: Seller Onboarding + Seller Portal (Products + Orders)

**Goal:** Sellers can complete the full onboarding journey (register → setup wizard → subscribe → list products). Seller order management (confirm/ship/deliver) is functional. GST invoice PDF generated on subscription.

**Duration:** 2026-07-28 → 2026-08-08
**Committable:** 28 SP
**Committed:** 27 SP

| ID | Title | Type | SP | Assignee | Status |
|----|-------|------|----|---------|--------|
| TASK-029 | Implement Celery email tasks for order notifications | Backend | 3 | Backend Dev | Pending |
| TASK-036 | Implement Razorpay subscription creation and webhook handlers | Backend | 5 | Backend Dev | Pending |
| TASK-037 | Implement GST invoice PDF generation with WeasyPrint | Backend | 3 | Backend Dev | Pending |
| TASK-038 | Implement payout bank account details with field encryption | Backend | 3 | Backend Dev | Pending |
| TASK-039 | Implement store setup wizard UI (SCR-011) | Frontend | 3 | Frontend Dev | Pending |
| TASK-040 | Implement seller subscription signup UI (SCR-012) | Frontend | 2 | Frontend Dev | Pending |
| TASK-042 | Implement low-stock alert Celery task | Backend | 2 | Backend Dev | Pending |
| TASK-043 | Implement seller product management UI (SCR-003) | Frontend | 3 | Frontend Dev | Pending |
| TASK-045 | Implement seller order management API endpoints | Backend | 3 | Backend Dev | Pending |
| TASK-046 | Implement seller order management UI (SCR-014) | Frontend | 3 | Frontend Dev | Pending |
| TASK-047 | Write seller order fulfillment tests | Test | 2 | Backend Dev | Pending |
| TASK-078 | Write seller onboarding backend tests | Test | 2 | Backend Dev | Pending |
| TASK-079 | Write product management backend tests | Test | 2 | Backend Dev | Pending |

**Sprint 5 SP Total:** 3+5+3+3+3+2+2+3+3+3+2+2+2 = **37 SP raw**

*Sprint 5 is the highest volume sprint due to seller onboarding complexity. Backend dev handles: TASK-029, TASK-036, TASK-037, TASK-038, TASK-042, TASK-045, TASK-047, TASK-078, TASK-079 = 25 SP (over 10 days = 2.5 SP/day — tight but achievable with parallel test writing). Frontend dev handles: TASK-039, TASK-040, TASK-043, TASK-044, TASK-046 = 14 SP. TASK-044 (3 SP product form UI) may move to Sprint 6 if capacity is reached. Flag at Sprint 5 day 5 checkpoint.*

**Sprint 5 Done Criteria:**
- Seller completes registration → store wizard → subscription payment → can list products
- GST invoice PDF generated and downloadable on subscription confirmation
- Bank account number encrypted at rest (verified via direct DB query)
- Seller can confirm, ship, and deliver orders via dashboard
- Buyer receives shipped email with AWB within 60s of seller action

---

## Sprint 6: Payouts, Settlement, Analytics, Subscription Management

**Goal:** Weekly payout job executes and disburses seller earnings via Razorpay Payouts API. Seller analytics dashboard shows revenue and top products. Subscription status management and invoice download complete.

**Duration:** 2026-08-11 → 2026-08-22
**Committable:** 28 SP
**Committed:** 27 SP

| ID | Title | Type | SP | Assignee | Status |
|----|-------|------|----|---------|--------|
| TASK-044 | Implement add and edit product form UI | Frontend | 3 | Frontend Dev | Pending |
| TASK-048 | Implement settlement ledger and payout service | Backend | 2 | Backend Dev | Pending |
| TASK-049 | Implement weekly Celery Beat payout job | Backend | 5 | Backend Dev | Pending |
| TASK-050 | Implement payout history API | Backend | 3 | Backend Dev | Pending |
| TASK-051 | Build seller payout history frontend page (SCR-015) | Frontend | 2 | Frontend Dev | Pending |
| TASK-052 | Write payout and settlement tests | Test | 2 | Backend Dev | Pending |
| TASK-053 | Implement subscription status and invoice API endpoints | Backend | 2 | Backend Dev | Pending |
| TASK-054 | Build subscription management UI (SCR-016) | Frontend | 2 | Frontend Dev | Pending |
| TASK-055 | Implement bank account update with payout hold logic | Backend | 2 | Backend Dev | Pending |
| TASK-056 | Build seller store settings UI (SCR-017) | Frontend | 3 | Frontend Dev | Pending |
| TASK-057 | Implement seller analytics API endpoints | Backend | 2 | Backend Dev | Pending |
| TASK-058 | Implement seller order CSV export | Backend | 2 | Backend Dev | Pending |
| TASK-059 | Build seller dashboard home UI (SCR-013) | Frontend | 3 | Frontend Dev | Pending |

**Sprint 6 SP Total:** 3+2+5+3+2+2+2+2+2+3+2+2+3 = **33 SP raw / 27 SP committed**

*(TASK-059 may move to Sprint 7 if Sprint 6 backend payout work runs over.)*

**Sprint 6 Done Criteria:**
- Weekly payout Celery Beat job executes on Monday 09:00 IST schedule
- Razorpay Payouts API called for each eligible seller with correct net amount
- Payout email dispatched to seller on initiation
- Suspended sellers excluded from payout; balance preserved
- Seller analytics dashboard shows correct 7d/30d metrics
- Payout history page accessible in seller portal

---

## Sprint 7: Admin Portal + Buyer Order History

**Goal:** Admin can approve/reject sellers and products, suspend/reactivate sellers, and view platform metrics. Registered buyers can view their order history.

**Duration:** 2026-08-25 → 2026-09-05
**Committable:** 28 SP
**Committed:** 26 SP

| ID | Title | Type | SP | Assignee | Status |
|----|-------|------|----|---------|--------|
| TASK-059 | Build seller dashboard home UI (SCR-013) | Frontend | 3 | Frontend Dev | Pending |
| TASK-060 | Implement admin seller management API endpoints | Backend | 3 | Backend Dev | Pending |
| TASK-061 | Implement admin product approval API endpoints | Backend | 3 | Backend Dev | Pending |
| TASK-062 | Implement admin dashboard metrics API | Backend | 2 | Backend Dev | Pending |
| TASK-063 | Build admin login UI (SCR-018) | Frontend | 2 | Frontend Dev | Pending |
| TASK-064 | Build admin seller approval queue UI (SCR-019) | Frontend | 3 | Frontend Dev | Pending |
| TASK-065 | Build admin product approval queue UI (SCR-020) | Frontend | 3 | Frontend Dev | Pending |
| TASK-066 | Build admin dashboard UI (SCR-021) | Frontend | 2 | Frontend Dev | Pending |
| TASK-067 | Write admin portal backend tests | Test | 2 | Backend Dev | Pending |
| TASK-068 | Implement buyer order history API endpoint | Backend | 2 | Backend Dev | Pending |
| TASK-069 | Build buyer order history UI (SCR-009) | Frontend | 2 | Frontend Dev | Pending |
| TASK-075 | Verify OpenAPI spec and generate final API documentation | Documentation | 2 | Backend Dev | Pending |

**Sprint 7 SP Total:** 3+3+3+2+2+3+3+2+2+2+2+2 = **29 SP raw / 26 SP committed**

*(TASK-059 moved from Sprint 6 if not completed. Remove from here if completed in Sprint 6.)*

**Sprint 7 Done Criteria:**
- Admin can approve and reject sellers via approval queue UI
- Admin can approve and reject products via product queue UI
- Admin suspension immediately hides all seller products from marketplace
- Registered buyer sees complete order history sorted by date
- OpenAPI spec validates with zero warnings (`python manage.py spectacular --validate`)

---

## Sprint 8: Pre-Launch Hardening

**Goal:** Production-ready: Redis caching, security headers, WCAG 2.1 AA accessibility, CI/CD deployment to ECS Fargate, test coverage ≥ 80% across all apps, and load test confirming 1,000 concurrent users at P95 < 200ms.

**Duration:** 2026-09-08 → 2026-09-26 (3 weeks — slightly extended for pre-launch quality gate)

*Note: Sprint 8 is extended to 3 weeks (15 working days) to accommodate the load test, accessibility audit, and ECS deployment setup which require environment access beyond code writing. This brings the total sprint run to 17 working weeks (8 sprints × 2 weeks + 1 extra week), completing 2026-09-26 — 5 weeks before the 2026-10-31 MVP deadline.*

**Committable:** 35 SP (3-week sprint, same daily rate)
**Committed:** 30 SP

| ID | Title | Type | SP | Assignee | Status |
|----|-------|------|----|---------|--------|
| TASK-070 | Add Redis product catalog cache with 1-hour TTL | Backend | 2 | Backend Dev | Pending |
| TASK-071 | Configure security headers and CORS policy | Backend | 2 | Full-Stack Lead | Pending |
| TASK-072 | Accessibility audit and remediation — buyer-facing screens | Frontend | 3 | Frontend Dev | Pending |
| TASK-073 | Configure GitHub Actions CD pipeline to ECS Fargate | DevOps | 5 | Full-Stack Lead | Pending |
| TASK-074 | Close test coverage gaps to meet 80% threshold | Test | 3 | Full-Stack Lead | Pending |
| TASK-075 | Verify OpenAPI spec (if not completed in Sprint 7) | Documentation | 2 | Backend Dev | Pending |
| TASK-076 | Conduct load test — 1,000 concurrent users | Test | 3 | Full-Stack Lead | Pending |

**Sprint 8 SP Total:** 2+2+3+5+3+2+3 = **20 SP committed** (well within 35 SP committable — intentionally light to accommodate ECS setup complexity and stakeholder UAT)

**Sprint 8 Done Criteria:**
- `docker push + ECS deploy` pipeline green for both staging and production
- P95 API response < 200ms under 1,000 concurrent user load test
- axe DevTools: zero critical/serious WCAG violations on all 8 buyer-facing screens
- `pytest --cov` and `jest --coverage` both report ≥ 80% across all apps
- Security headers present on all responses (verified via curl)
- Staging environment accessible at `staging.shopnest.in`

---

## Total Sprint Summary

| Sprint | Goal | Dates | Committed SP | Key Deliverables |
|--------|------|-------|------------|-----------------|
| Sprint 1 | Foundation | Jun 02–13 | 22 SP | Docker, schema, JWT auth |
| Sprint 2 | Auth + Browse API | Jun 16–27 | 24 SP | Registration UI, product API, homepage SSR |
| Sprint 3 | Browse UI + Cart | Jun 30–Jul 11 | 26 SP | 4 SSR pages, cart (guest + auth) |
| Sprint 4 | Checkout + Payment | Jul 14–25 | 27 SP | Razorpay webhook, order tracking, refund |
| Sprint 5 | Seller Portal | Jul 28–Aug 08 | 27 SP | Onboarding, products, orders |
| Sprint 6 | Payouts + Analytics | Aug 11–22 | 27 SP | Weekly payouts, seller analytics, subscription mgmt |
| Sprint 7 | Admin Portal + Buyer History | Aug 25–Sep 05 | 26 SP | Admin queues, buyer orders |
| Sprint 8 | Pre-Launch Hardening | Sep 08–26 | 20 SP | CD pipeline, load test, accessibility, coverage |
| **Total** | | **Jun 02 – Sep 26** | **199 SP** | **MVP complete 5 weeks before Oct 31 deadline** |

---

## Post-Sprint 8 Buffer (Sep 27 – Oct 31)

5-week buffer available for:
- Stakeholder UAT and bug fixes from UAT findings
- Razorpay production onboarding completion (if delayed)
- Legal review (OQ-005: Consumer Protection Rules 2020)
- GST compliance review (subscription invoices)
- Performance tuning from load test findings
- Production environment final configuration
- Beta seller onboarding (up to 10 sellers before public launch)

---

*Sprint plan is a living document. Update committed tasks at the start of each sprint planning session.*
*Velocity may be recalibrated after Sprint 2 retrospective using actual completed SP.*
