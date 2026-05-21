# Task Dependency Map — ShopNest
**Phase:** 06 — Task Breakdown & Sprint Planning
**Generated:** 2026-05-19
**Method:** Critical Path Method (CPM) — all genuine dependencies only

---

## 1. Critical Path

The critical path is the longest chain of dependent tasks from project start to MVP-ready. Any delay on this chain delays the MVP delivery date.


```
TASK-001 (Monorepo init)
  → TASK-003 (Django init)
    → TASK-005 (DB migrations — all 19 entities)
      → TASK-010 (JWT auth)
        → TASK-011 (Seller registration API)
          → TASK-015 (RBAC permissions)
            → TASK-017 (Product listing/search API)
              → TASK-023 (Cart API)
                → TASK-027 (Checkout initiate API)
                  → TASK-028 (Razorpay webhook handler) ← LONGEST CHAIN START
                    → TASK-029 (Celery email tasks)
                    → TASK-036 (Subscription webhooks)
                      → TASK-049 (Weekly payout job) ← PAYOUT CRITICAL PATH END
                    → TASK-045 (Seller order mgmt API)
                      → TASK-060 (Admin seller mgmt API)
                        → TASK-062 (Admin dashboard metrics) ← ADMIN CRITICAL PATH END
```

**MVP is ready when:**
1. Buyer can browse → add to cart → checkout → pay → receive email (TASK-028 + TASK-029)
2. Seller can list products → receive order → fulfill → receive payout (TASK-041 + TASK-045 + TASK-049)
3. Admin can approve sellers and products (TASK-060 + TASK-061)

**Estimated minimum sequential calendar time (single developer on critical path):** 14 weeks at critical path velocity.

---

## 2. Full Dependency Graph

### EPIC-001: Foundation (Sprint 1 — all parallel except where noted)

```
TASK-001 (Monorepo init)
  ├── TASK-002 (Docker Compose)         — parallel with TASK-003, TASK-004
  ├── TASK-003 (Django init)
  │     ├── TASK-005 (DB migrations)   — blocks EVERYTHING backend
  │     ├── TASK-006 (Celery setup)    — depends on TASK-005
  │     ├── TASK-007 (CI pipeline)     — parallel with TASK-005
  │     └── TASK-008 (Health + logging)
  └── TASK-004 (Next.js init)          — parallel with TASK-003
        └── TASK-007 (CI pipeline)     — also depends on TASK-004

TASK-002 + TASK-003 → TASK-009 (S3/CloudFront config)
```

**Parallelizable in Sprint 1:**
- TASK-003 (backend dev) runs parallel to TASK-004 (frontend dev)
- TASK-007 (CI config by full-stack lead) runs parallel to TASK-005 (DB migrations by backend dev)
- TASK-009 (S3 config by full-stack lead) runs parallel to TASK-006 (Celery by backend dev)

---

### EPIC-002: Authentication (Sprint 1–2)

```
TASK-005 (DB migrations)
  └── TASK-010 (JWT auth)
        ├── TASK-011 (Seller reg API)
        │     └── TASK-015 (RBAC permissions)
        │           └── TASK-082 (Admin queue API stub)
        ├── TASK-012 (Buyer reg API)
        │     └── TASK-015 (RBAC permissions)
        └── TASK-016 (Admin mgmt command)

TASK-010 → TASK-013 (Seller reg UI)   — Frontend parallel to TASK-014
TASK-010 → TASK-014 (Buyer auth UI)   — Frontend parallel to TASK-013
TASK-010 + TASK-011 + TASK-012 → TASK-077 (Auth tests)
TASK-011 → TASK-082 (Admin queue API stub)
```

**Parallelizable in Sprint 2:**
- Backend dev: TASK-015 + TASK-016 + TASK-077
- Frontend dev: TASK-013 + TASK-014 (sequential — same dev)
- Full-stack lead: TASK-009 cleanup + code reviews

---

### EPIC-003: Browse & Search (Sprint 2–3)

```
TASK-005 + TASK-015
  └── TASK-017 (Product listing + search API)
        ├── TASK-018 (Homepage SSR)
        │     └── TASK-019 (Category listing SSR)
        │           └── TASK-021 (Search results SSR)    — parallel with TASK-020
        └── TASK-020 (Product detail SSR)                — parallel with TASK-019

TASK-017 → TASK-022 (Browse/search tests)
TASK-017 → TASK-081 (Slug generation utility)
```

**Parallelizable in Sprint 2–3:**
- Backend dev: TASK-017 → TASK-022 → TASK-081
- Frontend dev: TASK-018 → TASK-019 + TASK-020 (sequential — same dev)
- Note: TASK-021 (Search UI) can start while TASK-020 (PDP) is in review

---

### EPIC-004: Shopping Cart (Sprint 3)

```
TASK-005 + TASK-010
  └── TASK-023 (Cart API — Redis)
        └── TASK-024 (Cart merge on login)
              └── TASK-026 (Cart tests)

TASK-023 → TASK-025 (Cart UI — panel)   — Frontend, parallel to TASK-024

TASK-084 (AuditLog signals)              — parallel to cart work, Sprint 3
```

**Parallelizable in Sprint 3:**
- Backend dev: TASK-023 → TASK-024 → TASK-026 + TASK-084 + TASK-081
- Frontend dev: TASK-021 (Search UI) → TASK-025 (Cart UI)

---

### EPIC-005: Checkout & Payment (Sprint 3–4 — CRITICAL PATH)

```
TASK-023 + TASK-005
  └── TASK-027 (Checkout initiate API)
        └── TASK-028 (Webhook handler — CRITICAL PATH CORE)
              ├── TASK-029 (Email Celery tasks)
              ├── TASK-030 (Order tracking token API)
              └── TASK-031 (Order cancellation + refund)
                    └── TASK-034 (Checkout + webhook integration tests)

TASK-027 + TASK-025 → TASK-032 (Checkout UI)
TASK-030 + TASK-031 → TASK-033 (Order tracking UI)
TASK-012 → TASK-080 (Registered buyer address pre-fill)
TASK-032 + TASK-080 → TASK-033 (Checkout UI complete)
```

**TASK-028 is the most critical task in the backlog.** It touches orders, stock, ledger, and emails simultaneously. It blocks:
- Seller order fulfillment (EPIC-008)
- Payout settlement (EPIC-009)
- Admin dashboard metrics (EPIC-012)

**Parallelizable in Sprint 3–4:**
- Backend dev: TASK-027 → TASK-028 (sequential, same dev, critical path)
- Frontend dev: TASK-025 (Cart UI) → TASK-032 (Checkout UI) → TASK-033 (Tracking UI)
- After TASK-028 merges: TASK-029 (email tasks) + TASK-030 (tracking) + TASK-031 (cancel) can parallelize

---

### EPIC-006: Seller Onboarding (Sprint 4–5)

```
TASK-005 + TASK-009 + TASK-015
  └── TASK-035 (Store setup API)
        └── TASK-039 (Store wizard UI)     — Frontend

TASK-028 + TASK-006 + TASK-015
  └── TASK-036 (Subscription webhooks)
        └── TASK-037 (GST invoice PDF)
        └── TASK-040 (Subscription UI)     — Frontend parallel to TASK-037

TASK-005 + TASK-015
  └── TASK-038 (Bank account encryption)
        └── TASK-078 (Onboarding tests)
```

**Parallelizable in Sprint 4–5:**
- Backend dev: TASK-035 → TASK-038 → TASK-036 → TASK-037
- Frontend dev: TASK-039 → TASK-040

---

### EPIC-007: Seller Product Management (Sprint 4–5)

```
TASK-017 + TASK-035
  └── TASK-041 (Product CRUD API)
        ├── TASK-042 (Low-stock alert task)
        │     └── TASK-079 (Product mgmt tests)
        ├── TASK-043 (Product list UI)
        │     └── TASK-044 (Product form UI)
        └── TASK-081 (Slug utility)        — already done by Sprint 3
```

**Parallelizable:**
- Backend dev: TASK-041 → TASK-042 → TASK-079
- Frontend dev: TASK-043 → TASK-044

---

### EPIC-008: Seller Order Fulfillment (Sprint 5)

```
TASK-028 + TASK-029
  └── TASK-045 (Seller order management API)
        ├── TASK-047 (Order fulfillment tests)
        └── TASK-046 (Order management UI)   — Frontend parallel to TASK-047
```

**Full dependency chain: TASK-001 → TASK-005 → TASK-010 → TASK-015 → TASK-028 → TASK-045**

---

### EPIC-009: Payouts & Settlement (Sprint 6 — CRITICAL PATH TAIL)

```
TASK-045
  └── TASK-048 (Settlement ledger service)
        └── TASK-049 (Weekly payout job)         — critical path to revenue
              └── TASK-052 (Payout tests)

TASK-048 + TASK-049
  └── TASK-050 (Payout history API)
        └── TASK-051 (Payout history UI)          — Frontend
```

**Parallelizable in Sprint 6:**
- Backend dev: TASK-048 → TASK-049 → TASK-050 → TASK-052 + TASK-053 + TASK-055 + TASK-057 + TASK-058
- Frontend dev: TASK-051 → TASK-054 → TASK-056 → TASK-059

---

### EPIC-012: Admin Portal (Sprint 7)

```
TASK-015 + TASK-017
  └── TASK-060 (Admin seller management API)
        └── TASK-067 (Admin tests)

TASK-041 + TASK-015
  └── TASK-061 (Admin product approval API)
        └── TASK-067 (Admin tests)

TASK-015 + TASK-045 + TASK-017
  └── TASK-062 (Admin dashboard metrics)

TASK-010 → TASK-063 (Admin login UI)
TASK-060 → TASK-064 (Admin seller queue UI)
TASK-061 → TASK-065 (Admin product queue UI)
TASK-062 → TASK-066 (Admin dashboard UI)
```

---

### EPIC-013: Buyer Account (Sprint 7)

```
TASK-028 + TASK-012
  └── TASK-068 (Buyer order history API)
        └── TASK-069 (Buyer order history UI)   — depends on TASK-014 (buyer auth UI)
```

---

### EPIC-014: Pre-Launch Hardening (Sprint 8)

```
TASK-017 → TASK-070 (Redis product cache)

TASK-003 + TASK-004 → TASK-071 (Security headers)

TASK-018 + TASK-019 + TASK-020 + TASK-021 + TASK-025 + TASK-032 + TASK-033 + TASK-069
  → TASK-072 (Accessibility audit)

TASK-007 + TASK-002 → TASK-073 (CD pipeline to ECS)

All prior test tasks → TASK-074 (Coverage gap closure)

All backend tasks → TASK-075 (OpenAPI spec verification)

TASK-073 → TASK-076 (Load test on staging)

TASK-084 → spread across Sprint 3 (no Sprint 8 dependency)
```

---

## 3. Parallelization Summary by Sprint

| Sprint | Backend Dev | Frontend Dev | Full-Stack Lead |
|--------|-------------|--------------|-----------------|
| 1 | TASK-003, TASK-005, TASK-006, TASK-008, TASK-010, TASK-011 | TASK-004 | TASK-001, TASK-002, TASK-007, TASK-009 |
| 2 | TASK-012, TASK-015, TASK-016, TASK-017, TASK-077, TASK-082, TASK-083 | TASK-013, TASK-014, TASK-018 | Code reviews + architecture support |
| 3 | TASK-019 API support, TASK-022, TASK-023, TASK-024, TASK-026, TASK-027, TASK-081, TASK-084 | TASK-019, TASK-020, TASK-021, TASK-025 | Code reviews |
| 4 | TASK-028, TASK-029, TASK-030, TASK-031, TASK-034, TASK-035, TASK-041, TASK-080 | TASK-032, TASK-033, TASK-039, TASK-043 | Code reviews + TASK-028 pair programming |
| 5 | TASK-036, TASK-037, TASK-038, TASK-042, TASK-045, TASK-047, TASK-078, TASK-079 | TASK-040, TASK-044, TASK-046 | Code reviews |
| 6 | TASK-048, TASK-049, TASK-050, TASK-052, TASK-053, TASK-055, TASK-057, TASK-058 | TASK-051, TASK-054, TASK-056, TASK-059 | Code reviews |
| 7 | TASK-060, TASK-061, TASK-062, TASK-067, TASK-068 | TASK-063, TASK-064, TASK-065, TASK-066, TASK-069 | Code reviews + TASK-075 |
| 8 | TASK-070, TASK-071 (backend portion), TASK-074 (backend), TASK-075 | TASK-072, TASK-074 (frontend) | TASK-071 (frontend), TASK-073, TASK-076 |

---

## 4. Complete Dependency Table

| Task | Depends On | Blocks |
|------|-----------|--------|
| TASK-001 | none | TASK-002, TASK-003, TASK-004 |
| TASK-002 | TASK-001 | TASK-009 |
| TASK-003 | TASK-001 | TASK-005, TASK-006, TASK-007, TASK-008 |
| TASK-004 | TASK-001 | TASK-007, TASK-013, TASK-014, TASK-018, TASK-019, TASK-020, TASK-021, TASK-025, TASK-032, TASK-033, TASK-039, TASK-040, TASK-043, TASK-044, TASK-046, TASK-051, TASK-054, TASK-056, TASK-059, TASK-063, TASK-064, TASK-065, TASK-066, TASK-069 |
| TASK-005 | TASK-003 | TASK-006, TASK-010, TASK-011, TASK-012, TASK-015, TASK-017, TASK-023, TASK-027, TASK-035, TASK-038, TASK-041, TASK-045, TASK-048, TASK-060, TASK-061, TASK-062, TASK-068 |
| TASK-006 | TASK-003, TASK-005 | TASK-028, TASK-029, TASK-036, TASK-042, TASK-049, TASK-083, TASK-084 |
| TASK-007 | TASK-003, TASK-004 | none (CI only) |
| TASK-008 | TASK-003 | none (infra only) |
| TASK-009 | TASK-002, TASK-003 | TASK-035, TASK-037, TASK-041 |
| TASK-010 | TASK-005 | TASK-011, TASK-012, TASK-013, TASK-014, TASK-015, TASK-016, TASK-063 |
| TASK-011 | TASK-010 | TASK-013, TASK-015, TASK-077, TASK-082 |
| TASK-012 | TASK-010 | TASK-014, TASK-015, TASK-068, TASK-069, TASK-077, TASK-080 |
| TASK-013 | TASK-004, TASK-011 | none |
| TASK-014 | TASK-004, TASK-012 | TASK-069 |
| TASK-015 | TASK-010, TASK-005 | TASK-017, TASK-023, TASK-035, TASK-038, TASK-041, TASK-045, TASK-048, TASK-060, TASK-061, TASK-062, TASK-068, TASK-082 |
| TASK-016 | TASK-010 | none |
| TASK-017 | TASK-005, TASK-015 | TASK-018, TASK-019, TASK-020, TASK-021, TASK-022, TASK-041, TASK-060, TASK-061, TASK-062, TASK-070 |
| TASK-018 | TASK-004, TASK-017 | TASK-019, TASK-021, TASK-072 |
| TASK-019 | TASK-004, TASK-017, TASK-018 | TASK-021, TASK-072 |
| TASK-020 | TASK-004, TASK-017 | TASK-072 |
| TASK-021 | TASK-004, TASK-017, TASK-018 | TASK-072 |
| TASK-022 | TASK-017 | none |
| TASK-023 | TASK-005, TASK-010 | TASK-024, TASK-025, TASK-026, TASK-027 |
| TASK-024 | TASK-023, TASK-010 | TASK-026 |
| TASK-025 | TASK-004, TASK-023 | TASK-032, TASK-072 |
| TASK-026 | TASK-023, TASK-024 | none |
| TASK-027 | TASK-023, TASK-005 | TASK-028, TASK-032 |
| TASK-028 | TASK-005, TASK-027, TASK-006 | TASK-029, TASK-030, TASK-031, TASK-034, TASK-036, TASK-045, TASK-048, TASK-068 |
| TASK-029 | TASK-006, TASK-028 | TASK-031, TASK-033, TASK-045, TASK-046 |
| TASK-030 | TASK-028, TASK-005 | TASK-033 |
| TASK-031 | TASK-028, TASK-029 | TASK-033, TASK-034 |
| TASK-032 | TASK-025, TASK-027 | TASK-033, TASK-072 |
| TASK-033 | TASK-004, TASK-030, TASK-031 | TASK-072 |
| TASK-034 | TASK-028, TASK-031 | none |
| TASK-035 | TASK-005, TASK-009, TASK-015 | TASK-039, TASK-041, TASK-078 |
| TASK-036 | TASK-028, TASK-006, TASK-015 | TASK-037, TASK-040, TASK-053, TASK-078 |
| TASK-037 | TASK-036, TASK-009 | TASK-053, TASK-078 |
| TASK-038 | TASK-005, TASK-015 | TASK-049, TASK-055, TASK-078 |
| TASK-039 | TASK-004, TASK-035 | none |
| TASK-040 | TASK-004, TASK-036 | none |
| TASK-041 | TASK-017, TASK-035 | TASK-042, TASK-043, TASK-044, TASK-061, TASK-079, TASK-081 |
| TASK-042 | TASK-028, TASK-041, TASK-006 | TASK-079 |
| TASK-043 | TASK-004, TASK-041 | TASK-044 |
| TASK-044 | TASK-004, TASK-041, TASK-043 | none |
| TASK-045 | TASK-028, TASK-029 | TASK-046, TASK-047, TASK-048, TASK-057, TASK-058, TASK-060, TASK-062, TASK-068 |
| TASK-046 | TASK-004, TASK-045 | none |
| TASK-047 | TASK-045 | none |
| TASK-048 | TASK-045, TASK-005 | TASK-049, TASK-050, TASK-052 |
| TASK-049 | TASK-048, TASK-006, TASK-038 | TASK-050, TASK-051, TASK-052, TASK-055 |
| TASK-050 | TASK-048, TASK-049 | TASK-051 |
| TASK-051 | TASK-004, TASK-050 | none |
| TASK-052 | TASK-049 | none |
| TASK-053 | TASK-036, TASK-037 | TASK-054 |
| TASK-054 | TASK-004, TASK-053 | none |
| TASK-055 | TASK-038, TASK-049 | TASK-056 |
| TASK-056 | TASK-004, TASK-035, TASK-055 | none |
| TASK-057 | TASK-045, TASK-005 | TASK-059 |
| TASK-058 | TASK-045 | none |
| TASK-059 | TASK-004, TASK-057, TASK-045 | none |
| TASK-060 | TASK-015, TASK-017 | TASK-064, TASK-067 |
| TASK-061 | TASK-041, TASK-015 | TASK-065, TASK-067 |
| TASK-062 | TASK-015, TASK-045, TASK-017 | TASK-066, TASK-067 |
| TASK-063 | TASK-004, TASK-010 | none |
| TASK-064 | TASK-004, TASK-060 | none |
| TASK-065 | TASK-004, TASK-061 | none |
| TASK-066 | TASK-004, TASK-062 | none |
| TASK-067 | TASK-060, TASK-061, TASK-062 | none |
| TASK-068 | TASK-028, TASK-012 | TASK-069 |
| TASK-069 | TASK-004, TASK-068, TASK-014 | TASK-072 |
| TASK-070 | TASK-017 | none |
| TASK-071 | TASK-003, TASK-004 | none |
| TASK-072 | TASK-018, TASK-019, TASK-020, TASK-021, TASK-025, TASK-032, TASK-033, TASK-069 | none |
| TASK-073 | TASK-007, TASK-002 | TASK-076 |
| TASK-074 | All prior test tasks | none |
| TASK-075 | All backend tasks | none |
| TASK-076 | TASK-073 | none |
| TASK-077 | TASK-010, TASK-011, TASK-012 | none |
| TASK-078 | TASK-035, TASK-036, TASK-037, TASK-038 | none |
| TASK-079 | TASK-041, TASK-042 | none |
| TASK-080 | TASK-012, TASK-032 | none |
| TASK-081 | TASK-017, TASK-041 | none |
| TASK-082 | TASK-011, TASK-015 | none |
| TASK-083 | TASK-005, TASK-006 | none |
| TASK-084 | TASK-005, TASK-006 | none |

---

## 5. Key Risk Points on Dependency Graph

| Risk Point | Task | Why It's High Risk |
|-----------|------|--------------------|
| **DB migration completeness** | TASK-005 | 19 entities; missing field or constraint blocks every subsequent backend task |
| **Webhook handler correctness** | TASK-028 | HMAC validation + idempotency + stock decrement + ledger — a bug here causes double-orders, overselling, or lost revenue |
| **Razorpay Payouts API enablement** | TASK-049 | Open question OQ (CLAUDE.md): Razorpay account must be enabled for Payouts API before this task can be tested in staging |
| **Field encryption key management** | TASK-038 | If encryption key is rotated mid-cycle without a migration plan, all existing bank account records become unreadable |
| **Celery Beat schedule accuracy** | TASK-049 | Monday 09:00 IST = Monday 03:30 UTC — must be tested in staging timezone; incorrect cron delays all payouts by a week |

---

## 6. ASCII Dependency Graph — Critical Path

```
Sprint 1
TASK-001 ─┬─► TASK-003 ─► TASK-005 ─► TASK-010 ─► TASK-011 ─► TASK-015 ─────────────────────────────┐
           │                                                                                            │
           ├─► TASK-004 ─────────────────────────────────────────────────────────────────────────────► │
           │                                                                                            │
           └─► TASK-002 ─► TASK-009 ──────────────────────────────────────────────────────────────────►│

Sprint 2                                                                                                │
                                         TASK-017 (Product API) ◄──────────────────────────────────────┘
                                             │
                              ┌──────────────┼──────────────┐
Sprint 2-3                    │              │              │
                          TASK-018      TASK-019        TASK-020
                          (Homepage)    (Category)      (PDP)
                              │
Sprint 3               TASK-023 (Cart API)
                            │
                        TASK-027 (Checkout init)
                            │
                        TASK-028 (Webhook handler) ◄─── CRITICAL PATH CORE ───
                        ┌───┼───────────────────┐
                    TASK-029  TASK-030       TASK-036
                    (Emails)  (Tracking)     (Subscriptions)
                                                │
Sprint 6                                   TASK-048 (Ledger)
                                                │
                                           TASK-049 (Payout job) ◄── REVENUE CRITICAL PATH END
```

---

*Dependency map generated from task acceptance criteria cross-reference.*
*Genuine blocking dependencies only — assumed sequential conventions excluded.*
*Update this file whenever a new task is added or a dependency is discovered mid-sprint (Rule 6).*
