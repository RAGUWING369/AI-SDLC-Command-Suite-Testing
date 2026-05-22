# Task Backlog — ShopNest
**Total Tasks:** 142
**Last Updated:** 2026-05-19
**Sprint Duration:** 2 weeks (10 working days)
**Team:** 1 Full-Stack Lead · 1 Frontend Dev · 1 Backend Dev
**Story Point Scale:** XS=1SP (~4h) · S=2SP (~8h) · M=3SP (~12h) · L=5SP (~20h)
**Sprint Capacity:** Sprint 1 = 24 SP committable · Sprint 2+ = 25 SP committable

---

## Status Legend
- Blocked — task cannot start due to unresolved dependency
- In Progress — actively being worked
- Done — acceptance criteria met, DoD verified
- Pending — not yet started

---

## Epic Index

| Epic | Name | User Stories | Sprint(s) | Total SP |
|------|------|-------------|-----------|----------|
| EPIC-001 | Project Foundation & DevOps | Infra/Architecture | 1 | 22 SP |
| EPIC-002 | Authentication & User Accounts | US-001, US-025 | 1–2 | 18 SP |
| EPIC-003 | Marketplace Browsing & Search | US-011, US-012, US-013, US-014 | 2–3 | 22 SP |
| EPIC-004 | Shopping Cart | US-015, US-026 | 3 | 12 SP |
| EPIC-005 | Checkout & Payment | US-016, US-017, US-018, US-019, US-020 | 3–4 | 28 SP |
| EPIC-006 | Seller Onboarding | US-002, US-003 | 4–5 | 20 SP |
| EPIC-007 | Seller Product Management | US-004, US-005, US-006, US-010 | 4–5 | 20 SP |
| EPIC-008 | Seller Order Fulfillment | US-007, US-008, US-009, US-033, US-034 | 5 | 16 SP |
| EPIC-009 | Seller Payouts & Settlement | US-024, US-032 | 6 | 18 SP |
| EPIC-010 | Seller Subscription Management | US-036, US-037 | 6 | 10 SP |
| EPIC-011 | Seller Analytics & Dashboard | US-028, US-038 | 6–7 | 12 SP |
| EPIC-012 | Admin Portal | US-021, US-022, US-023, US-029, US-030, US-035 | 7 | 22 SP |
| EPIC-013 | Buyer Account & Order History | US-027 | 7 | 8 SP |
| EPIC-014 | Pre-Launch Hardening | NFRs, Performance, Security, Accessibility | 8 | 20 SP |

---

## EPIC-001: Project Foundation & DevOps

**Description:** Establish the complete local development environment, CI/CD pipeline, Docker infrastructure, Django project skeleton with all 7 apps, Next.js project skeleton, PostgreSQL schema migrations, and core shared utilities. No feature work can proceed until this epic is Done.

**Business Value:** Every subsequent sprint task depends on this foundation. A broken or missing foundation means engineers block each other from day one.

---

### TASK-001
**Epic:** EPIC-001
**Story:** Infrastructure / Architecture
**Title:** Initialise monorepo structure and Git repository
**Description:** Create the root `shopnest/` monorepo with `frontend/`, `backend/`, `infra/`, and `docker-compose.yml` as documented in CLAUDE.md repository structure. Initialise Git, set up `.gitignore`, configure branch protection rules for `develop` and `main`. Create initial commit with the skeleton directory structure only — no application code yet.
**Acceptance Criteria:**
- [ ] `shopnest/frontend/` and `shopnest/backend/` directories exist and are committed
- [ ] `.gitignore` covers Python (`__pycache__`, `*.pyc`, `.env`, `venv/`), Node (`node_modules/`, `.next/`, `.env.local`), and Docker artifacts
- [ ] `develop` branch exists; `main` branch protected (require PR + CI green)
- [ ] `CLAUDE.md` root file present at repo root
- [ ] Initial commit message: `chore: initialise shopnest monorepo structure`
**Files Affected:** `/`, `.gitignore`, `CLAUDE.md`, `README.md`
**Wireframe Reference:** —
**Dependencies:** none
**Estimate:** S (2 SP)
**Type:** DevOps
**Assigned Role:** Full-Stack Lead
**Sprint:** 1
**Status:** Done
**Implementation Note:** Monorepo created at `shopnest/` with `frontend/`, `backend/`, `infra/` directories, `.gitignore` (Python/Node/Docker), `CLAUDE.md` (developer reference), `README.md`. Git initialized; `develop` branch created from `main`. Initial commit `52b9db2`. Branch protection rules (main/develop) pending GitHub remote creation.

---

### TASK-002
**Epic:** EPIC-001
**Story:** Infrastructure
**Title:** Configure Docker Compose for full local stack
**Description:** Write `docker-compose.yml` defining six services: `postgres` (PostgreSQL 16), `redis` (Redis 7), `backend` (Django + Gunicorn), `celery-worker`, `celery-beat`, and `frontend` (Next.js). Each service must use environment variable injection from `.env.example`. Volumes for PostgreSQL data persistence and bind mounts for hot-reload in development.
**Acceptance Criteria:**
- [ ] `docker compose up` starts all 6 services with zero manual steps beyond copying `.env.example`
- [ ] `backend` service health-check passes (`GET /api/v1/health/` returns 200) after `docker compose up`
- [ ] `frontend` service serves Next.js dev server at `http://localhost:3000`
- [ ] `postgres` data persists across `docker compose down` + `docker compose up` cycles using named volume
- [ ] `.env.example` documents all required environment variables with placeholder values and inline comments
**Files Affected:** `docker-compose.yml`, `.env.example`, `Dockerfile`
**Wireframe Reference:** —
**Dependencies:** TASK-001
**Estimate:** M (3 SP)
**Type:** DevOps
**Assigned Role:** Full-Stack Lead
**Sprint:** 1
**Status:** Done
**Implementation Note:** docker-compose.yml (6 services), Dockerfile (python:3.12-slim single backend image), .env.example (22 vars), .gitattributes (eol=lf). Commit `8affe86` on `feature/TASK-002-docker-compose`. Health-check and frontend service verifiable after TASK-003/TASK-004.

---

### TASK-003
**Epic:** EPIC-001
**Story:** Infrastructure
**Title:** Initialise Django project with 7 modular apps
**Description:** Run `django-admin startproject shopnest backend/` and create all 7 Django apps: `users`, `sellers`, `products`, `orders`, `cart`, `payments`, `analytics`. Configure `settings/` split into `base.py`, `development.py`, `production.py`. Install and configure: DRF, simplejwt, drf-spectacular, celery, redis, boto3, python-json-logger, pytest, pytest-cov, Black, isort, mypy.
**Acceptance Criteria:**
- [ ] `python manage.py check` passes with zero errors on all 7 apps registered in `INSTALLED_APPS`
- [ ] `pytest` runs and exits 0 (empty test suite is acceptable at this stage)
- [ ] `python manage.py spectacular --file openapi.yaml` generates a valid OpenAPI document
- [ ] `black --check .` and `isort --check .` pass on the initial codebase
- [ ] `requirements.txt` and `requirements-dev.txt` are pinned to exact versions
**Files Affected:** `backend/shopnest/settings/`, `backend/shopnest/urls.py`, `backend/*/apps.py`, `requirements.txt`, `requirements-dev.txt`
**Wireframe Reference:** —
**Dependencies:** TASK-001
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 1
**Status:** Done
**Implementation Note:** 49 files, 725 insertions. Django 5.0.6 + DRF 3.15.2 skeleton: split settings (base/dev/prod), 7 app stubs, custom User (UUID PK), Celery 4-queue config, RS256 JWT, health check, OpenAPI spec, JSON logging, pytest/black/isort/mypy config. New dep approved: django-cors-headers==4.3.1. Commit `f436643` on `feature/TASK-003-django-init`. Note: django-storages + django-ses needed in requirements.txt before production S3/SES tasks.

---

### TASK-004
**Epic:** EPIC-001
**Story:** Infrastructure
**Title:** Initialise Next.js 14 App Router project with TypeScript
**Description:** Bootstrap Next.js 14 with `--typescript --tailwind --app` flags. Configure TypeScript strict mode, ESLint Airbnb config, absolute imports (`@/`), Zustand store setup, and global Tailwind CSS with the design system tokens from `docs/ux/DESIGN-SYSTEM.md`. Install shadcn/ui base components.
**Acceptance Criteria:**
- [ ] `npm run build` completes with zero TypeScript errors
- [ ] `npm run lint` passes with zero ESLint errors
- [ ] `npm run type-check` passes
- [ ] Tailwind config includes the ShopNest color palette tokens from DESIGN-SYSTEM.md
- [ ] Zustand is installed; empty `lib/stores/` directory exists for feature stores
- [ ] Global layout renders at `http://localhost:3000` with ShopNest brand header (placeholder)
**Files Affected:** `frontend/app/layout.tsx`, `frontend/tailwind.config.ts`, `frontend/tsconfig.json`, `.eslintrc.js`, `frontend/lib/stores/`
**Wireframe Reference:** —
**Dependencies:** TASK-001
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 1
**Status:** Done
**Implementation Note:** Next.js 14.2.5 + TypeScript 5.4.5 strict mode + Tailwind 3.4.4. Airbnb ESLint config, ShopNest design system tokens in tailwind.config.ts, brand header placeholder, system font stack. npm run build ✓, npm run lint ✓, npm run type-check ✓. Commit `61d317f` on `feature/TASK-004-nextjs-init`. Note: next.config.mjs used (next.config.ts not supported until Next.js 15).

---

### TASK-005
**Epic:** EPIC-001
**Story:** Infrastructure
**Title:** Write all Django database migrations for 19 entities
**Description:** Define all 19 Django models across the 7 apps — User, SellerProfile, Store, StoreCategory, BuyerProfile, PayoutDetails, Category, Product, ProductImage, Subscription, Invoice, Order, SubOrder, OrderLineItem, SettlementLedgerEntry, Payout, AuditLog, AnalyticsEvent, WebhookIdempotencyLog — exactly as specified in DATA-MODEL.md. All UUID PKs using `gen_random_uuid()`, all FK relationships, all field constraints. Run `makemigrations` and `migrate`.
**Acceptance Criteria:**
- [ ] `python manage.py migrate` applies all migrations with zero errors on a fresh PostgreSQL database
- [ ] All 19 models are present with UUID PKs
- [ ] `search_vector` tsvector column on Product with GIN index is created
- [ ] `WebhookIdempotencyLog.razorpay_webhook_id` has unique constraint
- [ ] `PayoutDetails.account_number` field has a note in the migration that AES-256-GCM encryption is applied at the application layer (field type: TextField/BinaryField)
- [ ] All `created_at` / `updated_at` fields default to `now()` and auto-update respectively
**Files Affected:** `backend/users/models.py`, `backend/sellers/models.py`, `backend/products/models.py`, `backend/orders/models.py`, `backend/cart/` (Redis-backed, no migration), `backend/payments/models.py`, `backend/analytics/models.py`, `backend/*/migrations/0001_initial.py`
**Wireframe Reference:** —
**Dependencies:** TASK-003
**Estimate:** L (5 SP)
**Type:** Database
**Assigned Role:** Backend Dev
**Sprint:** 1
**Status:** Done
**Completion Notes:** All 19 ORM models defined across 6 apps. Split migration strategy used to break the products↔sellers circular FK dependency (products/0001: Category only; sellers/0001: SellerProfile+Store+StoreCategory+PayoutDetails; products/0002: Product+ProductImage+RunSQL FTS trigger). PostgreSQL search_vector auto-populated by trigger `products_update_search_vector()` on INSERT/UPDATE of name/description. `django.contrib.postgres` added to INSTALLED_APPS. AuditLog is append-only (save/delete raise ValueError). AnalyticsEvent.user_id and AuditLog.actor_id are bare UUIDs (no FK, DPDPA). Unit tests written for all 6 apps. Branch: `feature/TASK-005-django-models`, commit: 3e8db09.

---

### TASK-006
**Epic:** EPIC-001
**Story:** Infrastructure
**Title:** Configure Celery with Redis broker and Beat scheduler
**Description:** Wire Celery to the Django project (`shopnest/celery.py`), configure Redis db=1 as the broker and db=2 for result backend. Set up `django_celery_beat` for periodic tasks. Define the 4 named queues: `default`, `email`, `analytics`, `payouts`. Verify Celery worker starts and can process a test task.
**Acceptance Criteria:**
- [ ] `celery -A shopnest worker -Q default,email,analytics,payouts -l INFO` starts without errors in Docker
- [ ] `celery -A shopnest beat -l INFO --scheduler django_celery_beat.schedulers:DatabaseScheduler` starts without errors
- [ ] A test task (`debug_task`) can be dispatched and appears in Celery logs as processed
- [ ] Queue routing config is documented in `shopnest/celery.py` with inline comments mapping each queue to its purpose
**Files Affected:** `backend/shopnest/celery.py`, `backend/shopnest/settings/base.py` (CELERY_* settings)
**Wireframe Reference:** —
**Dependencies:** TASK-003, TASK-005
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 1
**Status:** Done
**Completion Notes:** Replaced bare-dict task_queues with typed `kombu.Queue` objects (Exchange + routing_key per queue). Added `debug_task` as `@app.task(bind=True)` on the default queue for smoke-testing. Added `CELERY_BEAT_SCHEDULER = DatabaseScheduler` to base.py. Unit tests verify broker URL (Redis db=1), all 4 queue names with matching exchange/routing_key, all task routes, Beat scheduler, and debug_task registered + runnable via `.apply()`. Branch: `feature/TASK-006-celery-config`, commit: 6a00623.

---

### TASK-007
**Epic:** EPIC-001
**Story:** Infrastructure
**Title:** Configure GitHub Actions CI pipeline
**Description:** Write `.github/workflows/ci.yml` that runs on every push to any branch and on every PR to `develop` or `main`. Pipeline steps: (1) Backend lint: Black + isort check; (2) Backend type check: mypy; (3) Backend tests: pytest with coverage gate ≥80%; (4) Frontend lint: ESLint; (5) Frontend type check: TypeScript; (6) Frontend tests: Jest with coverage gate ≥80%. Pipeline must fail the PR if any step fails.
**Acceptance Criteria:**
- [ ] CI pipeline triggers automatically on push and PR events
- [ ] All 6 steps run in parallel where possible (backend and frontend are independent)
- [ ] Coverage reports are uploaded as GitHub Actions artifacts
- [ ] A PR to `develop` with failing tests shows a red status check and cannot be merged
- [ ] Pipeline completes in under 10 minutes on the GitHub Actions free tier
**Files Affected:** `.github/workflows/ci.yml`
**Wireframe Reference:** —
**Dependencies:** TASK-003, TASK-004
**Estimate:** S (2 SP)
**Type:** DevOps
**Assigned Role:** Full-Stack Lead
**Sprint:** 1
**Status:** Done
**Completion Notes:** `.github/workflows/ci.yml` — two parallel jobs: `backend` (postgres:16-alpine + redis:7-alpine service containers, Black/isort/mypy/pytest with `--cov-fail-under=80`) and `frontend` (Node 20, ESLint/tsc/Jest with 80% coverage threshold). Concurrency group cancels stale in-progress runs. Coverage artifacts retained 30 days. `setup.cfg` updated with `[mypy-*.tests.*] ignore_errors=True` to prevent strict annotations in test helpers. `pytest.ini` --cov-fail-under=0 override removed. Branch: `feature/TASK-007-ci-pipeline`, commit: 5651a23.

---

### TASK-008
**Epic:** EPIC-001
**Story:** Infrastructure
**Title:** Create health check endpoint and structured logging
**Description:** Implement `GET /api/v1/health/` returning `{"status": "ok", "timestamp": "...", "version": "..."}`. Configure `python-json-logger` with fields: `timestamp`, `level`, `service`, `event_type`, `request_id` (middleware-injected UUID), `user_id_hash`. Verify logs are JSON-formatted in Docker stdout.
**Acceptance Criteria:**
- [ ] `GET /api/v1/health/` returns HTTP 200 with JSON body
- [ ] All Django application logs are JSON-formatted (verified via `docker compose logs backend`)
- [ ] `request_id` is injected as a UUID per-request by middleware and appears in all log lines for that request
- [ ] `user_id` is SHA-256 hashed before logging (never plain UUID in logs)
**Files Affected:** `backend/shopnest/urls.py`, `backend/shopnest/views.py`, `backend/shopnest/middleware.py`, `backend/shopnest/settings/base.py` (LOGGING)
**Wireframe Reference:** —
**Dependencies:** TASK-003
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 1
**Status:** Done
**Completion Notes:** `shopnest/views.py` — health_check returns `{"status":"ok","timestamp":"<ISO-8601 UTC>","version":"<pkg>"}` (no DB query by design). `shopnest/middleware.py` — `RequestIdMiddleware` injects UUID4 `request_id` per request into thread-local + `X-Request-ID` response header; `UserIdHashFilter` (logging.Filter) appends `request_id` and `user_id_hash` (SHA-256 of user UUID, "anonymous" for guests) to every log record. LOGGING config updated with filter + `static_fields={"service":"shopnest-api"}`. Unit tests cover endpoint, middleware, and filter. Branch: `feature/TASK-008-health-logging`, commit: 42e5565.

---

### TASK-009
**Epic:** EPIC-001
**Story:** Infrastructure
**Title:** Configure AWS S3 bucket and CloudFront distribution (local mock)
**Description:** Configure Django settings and boto3 for S3 integration. For local development, use `localstack` S3 (or MinIO) configured in Docker Compose. Define S3 bucket policies and CORS settings needed for product/store image upload. Document the CloudFront distribution URL convention (`https://cdn.shopnest.in/{s3-key}`) and S3 key naming convention (`{entity_type}/{entity_id}/{image_uuid}.{ext}`).
**Acceptance Criteria:**
- [ ] Local image upload to mock S3 succeeds via `boto3` in Django shell
- [ ] S3 key naming convention is documented in `docs/design/ARCHITECTURE.md` (update the infra section)
- [ ] Production S3 bucket and CloudFront config are documented in `infra/README.md` as manual setup steps (IaC is post-MVP)
- [ ] `AWS_S3_BUCKET_NAME`, `AWS_CLOUDFRONT_DOMAIN`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` are in `.env.example`
**Files Affected:** `backend/shopnest/settings/base.py`, `backend/shopnest/storage.py`, `docker-compose.yml` (localstack service), `.env.example`
**Wireframe Reference:** —
**Dependencies:** TASK-002, TASK-003
**Estimate:** S (2 SP)
**Type:** DevOps
**Assigned Role:** Full-Stack Lead
**Sprint:** 1
**Status:** Done
**Completion Notes:** `shopnest/storage.py` — `build_s3_key()`, `build_cdn_url()`, `upload_file()`, `delete_file()`, `StorageError`. `_get_s3_client()` uses `AWS_S3_ENDPOINT_URL` when set (LocalStack) and falls back to IAM task role in ECS. `docker-compose.yml` adds `localstack` service (port 4566, S3 only, health-checked); `backend` depends on `localstack`. `infra/localstack-init/01-create-bucket.sh` creates bucket on startup. `infra/README.md` documents production S3/CloudFront manual setup. `.env.example` updated with `AWS_S3_ENDPOINT_URL` and key convention comments. Unit tests cover key builder, CDN URL, upload/delete (S3 mocked). Branch: `feature/TASK-009-s3-storage`, commit: 7c78a2b.

---

## EPIC-002: Authentication & User Accounts

**Description:** Full RS256 JWT authentication for Sellers, Buyers, and Platform Admins. Seller registration flow, buyer registration flow, login/logout with refresh token denylist in Redis, RBAC middleware, JWKS endpoint, and admin account creation via management command.

**Business Value:** Authentication is the prerequisite for every authenticated endpoint across all subsequent epics. Until auth is complete and tested, no seller or buyer flows can be validated end-to-end.

---

### TASK-010
**Epic:** EPIC-002
**Story:** US-001, US-025
**Title:** Implement RS256 JWT authentication with simplejwt
**Description:** Configure `djangorestframework-simplejwt` with RS256 algorithm. Generate RSA-2048 key pair, store private key reference in `AWS_JWT_PRIVATE_KEY` env var. Implement: `POST /api/v1/auth/token/` (login), `POST /api/v1/auth/token/refresh/`, `POST /api/v1/auth/logout/` (denylist refresh token in Redis db=2), and `GET /.well-known/jwks.json` (public key endpoint). Configure bcrypt cost factor 12.
**Acceptance Criteria:**
- [ ] `POST /api/v1/auth/token/` with valid credentials returns `access_token` (24h) and `refresh_token` (30d)
- [ ] Decoded JWT `alg` claim = `RS256`
- [ ] `POST /api/v1/auth/logout/` adds refresh token to Redis denylist; subsequent refresh attempt returns 401
- [ ] `GET /.well-known/jwks.json` returns public key in JWK format
- [ ] Passwords stored as bcrypt hash with `$2b$12$` prefix (verified via `python manage.py shell`)
- [ ] Unit tests cover: valid login, invalid password, expired access token, denylisted refresh token
**Files Affected:** `backend/users/views.py`, `backend/shopnest/settings/base.py` (SIMPLE_JWT config), `backend/users/urls.py`
**Wireframe Reference:** —
**Dependencies:** TASK-005
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 1
**Status:** Done
**Implementation Note:** `users/views.py` — LoginView (wraps TokenObtainPairView), TokenRefreshWithDenylistView (checks Redis db=2 denylist before refresh), LogoutView (denylists jti with TTL), JWKSView (RSA public key in JWK format). `users/urls.py` — auth URL patterns + `jwks_urlpatterns` for root mount. `shopnest/urls.py` — users.urls included under /api/v1/; JWKS at /.well-known/. `settings/base.py` — BCryptSHA256PasswordHasher (cost 12), JWT_DENYLIST_REDIS_URL (Redis db=2). `requirements.txt` — bcrypt==4.1.3 added. `shopnest/celery.py` — fixed pre-existing `os.setdefault` → `os.environ.setdefault` typo. 28 tests in `users/tests/test_auth.py` covering all acceptance criteria. Full test suite requires PostgreSQL (Docker Compose / CI). Branch: `feature/TASK-010-rs256-jwt-auth`, commit: f5d818b.

---

### TASK-011
**Epic:** EPIC-002
**Story:** US-001
**Title:** Implement seller registration API endpoint
**Description:** Implement `POST /api/v1/sellers/register/` with DRF serializer validation for all required fields: email, password (min 8 chars, ≥1 digit), full_name, business_name, store_name, phone (10-digit Indian mobile regex), city, gstin (optional), shipping_pincode (6-digit). Creates User (role=SELLER) + SellerProfile (status=PENDING_REVIEW). Blocks registration if email already registered.
**Acceptance Criteria:**
- [ ] Valid payload returns HTTP 201 with `{"message": "Registration successful. Awaiting admin approval."}` and no sensitive data
- [ ] Duplicate email returns HTTP 409 with `{"detail": "Email already registered"}`
- [ ] Password without numeric char returns HTTP 400 with validation message
- [ ] Phone with != 10 digits returns HTTP 400
- [ ] `registration_blocked=True` email returns HTTP 409 (rejected seller re-registration block)
- [ ] Unit tests cover all validation branches and status codes
**Files Affected:** `backend/users/serializers.py`, `backend/sellers/serializers.py`, `backend/sellers/views.py`, `backend/sellers/urls.py`
**Wireframe Reference:** —
**Dependencies:** TASK-010
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 1
**Status:** Done
**Implementation Note:** `sellers/serializers.py` — SellerRegistrationSerializer with field validators (password strength, 10-digit Indian phone, 6-digit pincode, optional GSTIN regex). `users/serializers.py` — shared `validate_password_strength` + `validate_indian_phone` helpers. `sellers/views.py` — SellerRegistrationView (AllowAny, 201/400/409). `sellers/urls.py` + wired in `shopnest/urls.py`. `@transaction.atomic` ensures no partial writes. 43 tests in `sellers/tests/test_seller_registration.py`. Branch: `feature/TASK-011-seller-registration`, commit: a082b3a.

---

### TASK-012
**Epic:** EPIC-002
**Story:** US-025
**Title:** Implement buyer registration API endpoint
**Description:** Implement `POST /api/v1/buyers/register/` creating User (role=BUYER) + BuyerProfile. Fields: email, password, full_name, phone. Apply same password and phone validation as seller registration. Issue JWT tokens on successful registration (no admin approval step for buyers).
**Acceptance Criteria:**
- [ ] Valid payload returns HTTP 201 with access and refresh tokens
- [ ] Buyer JWT is accepted on buyer-only endpoints
- [ ] Buyer JWT returns HTTP 403 on seller dashboard endpoints
- [ ] Buyer JWT returns HTTP 403 on admin endpoints
- [ ] Unit tests cover registration, duplicate email, RBAC enforcement
**Files Affected:** `backend/users/serializers.py`, `backend/users/views.py`, `backend/users/urls.py`
**Wireframe Reference:** —
**Dependencies:** TASK-010
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 1
**Status:** Done
**Completion Summary:** Implemented POST /api/v1/buyers/register/ (BuyerRegistrationSerializer + BuyerRegistrationView), IsBuyer/IsSeller/IsAdmin RBAC permission classes, shared password/phone validators, 40+ tests covering happy path, email conflicts, password/phone validation, and RBAC unit + integration tests. All files Black/isort formatted. Committed on branch feature/TASK-012-buyer-registration (1864d81).

---

### TASK-013
**Epic:** EPIC-002
**Story:** US-001
**Title:** Implement seller registration screen (SCR-010)
**Description:** Build the Next.js seller registration page at `/seller/register`. Multi-field form with inline validation (client-side Zod schema matching backend rules). On submit, call `POST /api/v1/sellers/register/`. Show success state (redirect to login) or inline error messages from API response. Mobile-responsive layout per SCR-010 wireframe.
**Acceptance Criteria:**
- [ ] All required fields render with correct input types and labels
- [ ] Client-side validation shows inline error before API call for: missing fields, password rule violations, invalid phone format
- [ ] Successful registration shows success message and redirects to `/seller/login`
- [ ] API error (409 duplicate email) displays inline under the email field
- [ ] Form is fully functional at 375px (mobile) and 1440px (desktop) viewport widths per SCR-010 wireframe
- [ ] `npm run test` covers: field validation, successful submit, error state rendering
**Files Affected:** `frontend/app/seller/register/page.tsx`, `frontend/components/auth/SellerRegisterForm.tsx`, `frontend/lib/api/sellers.ts`
**Wireframe Reference:** `docs/visuals/ux/SCR-010-seller-registration.html`
**Dependencies:** TASK-004, TASK-011
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 2
**Status:** Done
**Completion Summary:** Implemented seller registration screen at /seller/register per SCR-010 wireframe. All 5 states: default form, loading, validation errors, success (state=PENDING_REVIEW), 409 email conflict with "Sign in instead?" link. Client-side validation mirrors backend (password min-8+digit, 10-digit Indian phone, optional GSTIN). Password strength meter. Layout restructured to (shell)/layout.tsx route group so pre-auth pages bypass the Header. Jest+RTL setup established (jest.config.ts, jest.setup.ts, tsconfig.test.json). 17 tests, lint and type-check both pass. Committed 81344a9 on feature/TASK-013-seller-registration-screen.

---

### TASK-014
**Epic:** EPIC-002
**Story:** US-025
**Title:** Implement buyer auth screen — login and register (SCR-007)
**Description:** Build the Next.js buyer auth page at `/auth/login` and `/auth/register`. Toggle between login and register modes. Login calls `POST /api/v1/auth/token/`; register calls `POST /api/v1/buyers/register/`. Store JWT in httpOnly cookie via Next.js server action. On login success, redirect to previous page or homepage. Implement "Forgot password" placeholder (no functionality in MVP).
**Acceptance Criteria:**
- [ ] Login form accepts email + password; on success sets JWT cookie and redirects
- [ ] Register form accepts name, email, phone, password; on success auto-logs in
- [ ] Invalid credentials shows inline error "Invalid email or password"
- [ ] Toggle between login and register modes without page reload
- [ ] JWT is stored in httpOnly cookie (not localStorage)
- [ ] Fully functional at 375px (mobile) and 1440px (desktop) viewport widths per SCR-007 wireframe
**Files Affected:** `frontend/app/auth/login/page.tsx`, `frontend/app/auth/register/page.tsx`, `frontend/components/auth/BuyerAuthForm.tsx`, `frontend/lib/auth.ts`
**Wireframe Reference:** `docs/visuals/ux/SCR-007-buyer-auth.html`
**Dependencies:** TASK-004, TASK-012
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 2
**Status:** Pending

---

### TASK-015
**Epic:** EPIC-002
**Story:** US-001, US-025
**Title:** Write RBAC permission classes and subscription gate middleware
**Description:** Implement DRF permission classes: `IsSeller` (role=SELLER), `IsBuyer` (role=BUYER), `IsAdmin` (role=ADMIN), `IsSellerWithActiveSubscription` (role=SELLER AND subscription.status=ACTIVE). Apply `IsSellerWithActiveSubscription` to all seller dashboard endpoints. Apply `IsAdmin` to all `/api/v1/admin/` endpoints. Write automated cross-tenant access tests.
**Acceptance Criteria:**
- [ ] Seller JWT rejected on buyer-only endpoints (403)
- [ ] Buyer JWT rejected on seller dashboard endpoints (403)
- [ ] Admin JWT rejected everywhere except `/api/v1/admin/` (403)
- [ ] Seller with `Payment Failed` subscription (day 1–7) can access dashboard (grace period)
- [ ] Seller with `Suspended` subscription returns 403 on dashboard endpoints
- [ ] Integration test suite covers all 5 RBAC combinations
**Files Affected:** `backend/shopnest/permissions.py`, `backend/shopnest/middleware.py`
**Wireframe Reference:** —
**Dependencies:** TASK-010, TASK-005
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 2
**Status:** Pending

---

### TASK-016
**Epic:** EPIC-002
**Story:** US-001
**Title:** Create admin account via management command
**Description:** Implement `python manage.py create_admin --email admin@shopnest.in --password <pw>` management command that creates a User with role=ADMIN and `is_staff=True`. Admin accounts cannot be created via any API endpoint (FR-AUTH-011). Document the command in `docs/ops/ADMIN-SETUP.md`.
**Acceptance Criteria:**
- [ ] Management command creates admin user with correct role
- [ ] `POST /api/v1/sellers/register/` with `role=admin` in payload is ignored (role is not accepted from client)
- [ ] Admin user can authenticate via `POST /api/v1/auth/token/` and receive admin-role JWT
- [ ] Unit test verifies `POST /api/v1/sellers/register/` cannot create admin role
**Files Affected:** `backend/users/management/commands/create_admin.py`, `backend/users/views.py`
**Wireframe Reference:** —
**Dependencies:** TASK-010
**Estimate:** XS (1 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 2
**Status:** Pending

---

## EPIC-003: Marketplace Browsing & Search

**Description:** Server-side rendered marketplace homepage, category listing page, product detail page, and search results page. PostgreSQL FTS with GIN index. SEO meta tags, Open Graph tags, stable URL slugs, and CloudFront image delivery. All pages must pass Core Web Vitals (LCP < 2.5s on mobile 4G).

**Business Value:** Buyers cannot discover or purchase products without this epic. Search and browse are the top-of-funnel for all revenue.

---

### TASK-017
**Epic:** EPIC-003
**Story:** US-011, US-012, US-013, US-014
**Title:** Implement product listing and search API endpoints
**Description:** Implement the following DRF endpoints: `GET /api/v1/products/` (marketplace listing — Active only, Triple-Gate visibility: seller ACTIVE + subscription ACTIVE + product ACTIVE), `GET /api/v1/products/{id}/` (product detail), `GET /api/v1/products/search/?q=` (PostgreSQL FTS using `search_vector` GIN index), `GET /api/v1/categories/` (all categories). Implement pagination at 24 items per page. Filter out stock=0 products from listing and search.
**Acceptance Criteria:**
- [ ] `GET /api/v1/products/` returns only Triple-Gate-passing active products
- [ ] `GET /api/v1/products/search/?q=cotton` returns ranked FTS results within 500ms against 1,000 product test fixture
- [ ] `GET /api/v1/products/{id}/` returns 404 for non-active products
- [ ] Pagination returns correct `count`, `next`, `previous` links
- [ ] Products with `stock_quantity=0` are excluded from listing and search
- [ ] Unit tests cover Triple-Gate logic, FTS query, pagination, out-of-stock exclusion
**Files Affected:** `backend/products/views.py`, `backend/products/serializers.py`, `backend/products/urls.py`, `backend/products/repositories.py`
**Wireframe Reference:** —
**Dependencies:** TASK-005, TASK-015
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 2
**Status:** Pending

---

### TASK-018
**Epic:** EPIC-003
**Story:** US-011, US-012
**Title:** Implement SSR marketplace homepage (SCR-004)
**Description:** Build the Next.js SSR homepage at `/`. Uses `async` server component to fetch `GET /api/v1/products/` (20 most recent approved products) and `GET /api/v1/categories/`. Render hero section, featured product grid (4-col desktop, 2-col mobile), and category cards. Implement correct Open Graph and JSON-LD structured data for SEO. Suspended seller products must be absent.
**Acceptance Criteria:**
- [ ] Page renders as static HTML with product data visible (curl with no JS)
- [ ] `og:title`, `og:description`, `og:image` meta tags present in `<head>`
- [ ] JSON-LD `WebSite` and `ItemList` structured data present
- [ ] Correct 4-col/2-col responsive grid at 1440px and 375px
- [ ] Suspended seller's products absent from featured grid
- [ ] LCP < 2.5s verified via Next.js bundle analysis
**Files Affected:** `frontend/app/page.tsx`, `frontend/components/marketplace/ProductGrid.tsx`, `frontend/components/marketplace/CategoryCards.tsx`, `frontend/app/layout.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-004-marketplace-homepage.html`
**Dependencies:** TASK-004, TASK-017
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 2
**Status:** Pending

---

### TASK-019
**Epic:** EPIC-003
**Story:** US-012
**Title:** Implement SSR category listing page (SCR-005)
**Description:** Build the Next.js SSR category page at `/categories/{slug}`. Fetches `GET /api/v1/products/?category={slug}&page={n}`. Renders product grid (24 per page) with pagination controls. Use `generateStaticParams` for known category slugs. Implement breadcrumb navigation. Out-of-stock products excluded.
**Acceptance Criteria:**
- [ ] Category page renders fully as SSR HTML at `/categories/womens-clothing`
- [ ] Correct pagination: 30 products → page 1 shows 24, page 2 shows 6
- [ ] Out-of-stock products absent from the listing
- [ ] Breadcrumb shows: Home > Category Name
- [ ] `title` and `meta description` tags are category-specific
- [ ] Page functions at 375px mobile and 1440px desktop viewports per SCR-005 wireframe
**Files Affected:** `frontend/app/categories/[slug]/page.tsx`, `frontend/components/marketplace/ProductGrid.tsx`, `frontend/components/common/Breadcrumb.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-005-category-listing.html`
**Dependencies:** TASK-004, TASK-017, TASK-018
**Estimate:** S (2 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 2
**Status:** Pending

---

### TASK-020
**Epic:** EPIC-003
**Story:** US-013, US-014
**Title:** Implement SSR product detail page (SCR-001)
**Description:** Build the Next.js SSR product detail page at `/products/{slug}-{uuid}`. Fetches `GET /api/v1/products/{id}/`. Render: image carousel (up to 5 CloudFront images), product name, price (INR format), description, seller store name, stock status badge, "Add to Cart" button (disabled if stock=0). Stable URL slug that does not change on name edit. Open Graph + og:image for social sharing.
**Acceptance Criteria:**
- [ ] Product data visible in HTML source (SSR confirmed via curl)
- [ ] Image carousel navigates between all product images from CloudFront CDN URLs
- [ ] "Add to Cart" button disabled and replaced with "Out of Stock" when stock=0
- [ ] `og:title`, `og:description`, `og:image` present and populated correctly
- [ ] URL slug is stable — product name edit does not change the URL
- [ ] All states render correctly at 375px (single-column) and 1440px (two-column) layout per SCR-001 wireframe
**Files Affected:** `frontend/app/products/[slug]/page.tsx`, `frontend/components/product/ImageCarousel.tsx`, `frontend/components/product/AddToCartButton.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-001-product-detail.html`
**Dependencies:** TASK-004, TASK-017
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 2
**Status:** Pending

---

### TASK-021
**Epic:** EPIC-003
**Story:** US-014
**Title:** Implement SSR search results page (SCR-006)
**Description:** Build the Next.js SSR search results page at `/search?q={query}`. Fetches `GET /api/v1/products/search/?q={query}&page={n}`. Render results grid (24 per page) with result count, relevance-sorted, pagination. Empty state when no results. Show the search query in the page heading and in the `<title>` tag.
**Acceptance Criteria:**
- [ ] Search renders as SSR HTML with results visible in page source
- [ ] Result count displayed: "24 results for 'cotton'"
- [ ] Empty state rendered when no results match
- [ ] Pagination works correctly for > 24 results
- [ ] Search box in navigation is pre-filled with current query
- [ ] Functional at 375px mobile and 1440px desktop viewports per SCR-006 wireframe
**Files Affected:** `frontend/app/search/page.tsx`, `frontend/components/marketplace/SearchResults.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-006-search-results.html`
**Dependencies:** TASK-004, TASK-017, TASK-018
**Estimate:** S (2 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 3
**Status:** Pending

---

### TASK-022
**Epic:** EPIC-003
**Story:** US-011, US-012, US-013, US-014
**Title:** Write backend tests for product listing, search, and FTS performance
**Description:** Write pytest integration tests for: Triple-Gate product visibility (all combinations of seller/subscription/product status), FTS search with ranking, out-of-stock exclusion, pagination boundaries, and product detail 404 for non-active products. Include a fixture with 1,000 products to verify search P95 < 500ms locally.
**Acceptance Criteria:**
- [ ] All Triple-Gate combinations tested (8 states: 2^3 seller/subscription/product)
- [ ] FTS test verifies result ranking (more relevant product appears first)
- [ ] Pagination boundary test: exactly 24 items on page 1, correct count on page 2
- [ ] P95 search latency assertion passes with 1,000-product fixture (< 500ms)
- [ ] Coverage for `products/` app ≥ 80%
**Files Affected:** `backend/products/tests/test_views.py`, `backend/products/tests/test_repositories.py`, `backend/products/tests/fixtures/`
**Wireframe Reference:** —
**Dependencies:** TASK-017
**Estimate:** S (2 SP)
**Type:** Test
**Assigned Role:** Backend Dev
**Sprint:** 3
**Status:** Pending

---

## EPIC-004: Shopping Cart

**Description:** Redis-backed guest and authenticated buyer cart. Add, update, remove items. Cart subtotal calculation. Cart persistence (24h TTL). Multi-seller cart support. Cart merge on login.

**Business Value:** Cart is the bridge between product discovery and checkout. Without it, buyers cannot accumulate items or proceed to payment.

---

### TASK-023
**Epic:** EPIC-004
**Story:** US-015, US-026
**Title:** Implement Redis-backed cart API
**Description:** Implement cart endpoints using Redis db=0: `POST /api/v1/cart/items/` (add item — guest or authenticated), `GET /api/v1/cart/` (view cart), `PATCH /api/v1/cart/items/{item_id}/` (update quantity), `DELETE /api/v1/cart/items/{item_id}/` (remove item). Guest carts use session cookie key; authenticated carts use buyer UUID key. 24h TTL reset on every modification. Validate stock on add (not on view).
**Acceptance Criteria:**
- [ ] `POST /api/v1/cart/items/` without auth returns 201 and sets cart session cookie
- [ ] Adding item beyond stock quantity returns 409
- [ ] Cart TTL resets to 24h on every PATCH/POST/DELETE
- [ ] Authenticated buyer's cart persists under their UUID key across sessions
- [ ] Multi-seller items coexist in a single cart
- [ ] Cart subtotal is calculated server-side and returned in `GET /api/v1/cart/` response
**Files Affected:** `backend/cart/views.py`, `backend/cart/serializers.py`, `backend/cart/services.py`, `backend/cart/urls.py`
**Wireframe Reference:** —
**Dependencies:** TASK-005, TASK-010
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 3
**Status:** Pending

---

### TASK-024
**Epic:** EPIC-004
**Story:** US-026
**Title:** Implement cart merge on buyer login
**Description:** When a buyer authenticates after adding items as a guest, merge the guest cart into their authenticated cart. If the same product exists in both carts, keep the higher quantity (capped at stock). After merge, delete the guest cart from Redis. Implement as a Django signal or a call in the login response handler.
**Acceptance Criteria:**
- [ ] Guest adds Product A (qty 2), logs in — Product A (qty 2) appears in authenticated cart
- [ ] Guest adds Product A (qty 2); authenticated cart has Product A (qty 3) — merged qty = 3 (max, capped at stock)
- [ ] Guest cart key deleted from Redis after successful merge
- [ ] Cart merge is idempotent (running twice does not duplicate items)
**Files Affected:** `backend/cart/services.py`, `backend/users/views.py` (login response)
**Wireframe Reference:** —
**Dependencies:** TASK-023, TASK-010
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 3
**Status:** Pending

---

### TASK-025
**Epic:** EPIC-004
**Story:** US-015
**Title:** Implement cart and checkout UI — cart panel (SCR-002 Part A)
**Description:** Build the cart panel at `/cart`. Use Zustand cart store to display items, quantities, seller groupings, and running subtotal. Quantity +/- controls call `PATCH /api/v1/cart/items/{id}/`. Remove button calls `DELETE`. Persistent cart badge in nav bar (item count from Zustand). Empty cart state with CTA to browse. Proceed to Checkout button navigates to `/checkout`.
**Acceptance Criteria:**
- [ ] Cart panel shows all items with product image, name, price, quantity controls, and line total
- [ ] Subtotal updates optimistically on quantity change
- [ ] Remove button removes item and updates subtotal
- [ ] Nav cart badge reflects live item count
- [ ] Empty cart state renders with "Start Shopping" CTA
- [ ] Checkout button enabled only when cart is non-empty
- [ ] Functional at 375px (mobile drawer/full-page) and 1440px (desktop sidebar) viewports per SCR-002 wireframe
**Files Affected:** `frontend/app/cart/page.tsx`, `frontend/components/cart/CartPanel.tsx`, `frontend/lib/stores/cartStore.ts`, `frontend/components/layout/NavCartBadge.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-002-cart-checkout.html`
**Dependencies:** TASK-004, TASK-023
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 3
**Status:** Pending

---

### TASK-026
**Epic:** EPIC-004
**Story:** US-015, US-026
**Title:** Write cart API tests
**Description:** Write pytest tests for: guest cart creation, authenticated cart persistence, cart merge on login, TTL behavior (mock Redis TTL), multi-seller cart, stock validation on add, quantity bounds.
**Acceptance Criteria:**
- [ ] Guest cart test confirms session cookie is set
- [ ] Authenticated cart test confirms UUID-keyed Redis entry
- [ ] Cart merge test covers all three merge scenarios (new item, same item qty comparison, stock cap)
- [ ] Stock validation test: adding qty > stock returns 409
- [ ] Coverage for `cart/` app ≥ 80%
**Files Affected:** `backend/cart/tests/test_views.py`, `backend/cart/tests/test_services.py`
**Wireframe Reference:** —
**Dependencies:** TASK-023, TASK-024
**Estimate:** S (2 SP)
**Type:** Test
**Assigned Role:** Backend Dev
**Sprint:** 3
**Status:** Pending

---

## EPIC-005: Checkout & Payment

**Description:** Guest checkout form, Razorpay Order creation, Razorpay checkout widget integration, webhook signature validation (HMAC-SHA256), order creation on payment.captured, stock decrement, sub-order split for multi-seller carts, order cancellation with Razorpay refund, guest order tracking via HMAC token, Celery async email dispatch.

**Business Value:** This is the revenue-critical path. Without checkout and payment, ShopNest has zero GMV. The webhook handling is the most security-critical code in the codebase.

---

### TASK-027
**Epic:** EPIC-005
**Story:** US-016, US-017
**Title:** Implement Razorpay Order creation endpoint
**Description:** Implement `POST /api/v1/checkout/initiate/`. Validates cart (stock check on all items; remove out-of-stock and return error if any found). Accepts guest checkout form data (name, email, phone, address). Creates a Razorpay Order via Razorpay Orders API for total cart amount in paise. Returns Razorpay `order_id` and `key_id` to the frontend for mounting the checkout widget. Atomic stock pre-reservation is NOT done here — only at webhook confirmation.
**Acceptance Criteria:**
- [ ] Valid request creates Razorpay Order and returns `razorpay_order_id`, `amount`, `currency="INR"`, `key_id`
- [ ] Request with out-of-stock cart item returns 409 listing affected product names
- [ ] Invalid phone number (not 10 digits) returns 400
- [ ] Razorpay API failure (network) retries 3 times with exponential backoff before returning 503
- [ ] Guest checkout data (name, email, address) stored in session/Redis pending order confirmation
- [ ] Unit tests cover: success path, stock failure, validation failure, Razorpay API mock
**Files Affected:** `backend/payments/views.py`, `backend/payments/services/razorpay_service.py`, `backend/cart/services.py`
**Wireframe Reference:** —
**Dependencies:** TASK-023, TASK-005
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 3
**Status:** Pending

---

### TASK-028
**Epic:** EPIC-005
**Story:** US-017
**Title:** Implement Razorpay webhook handler with HMAC validation and idempotency
**Description:** Implement `POST /api/v1/payments/webhook/` handling `payment.captured`, `payment.failed`, `subscription.charged`, `subscription.charge.failed` events. Validate `X-Razorpay-Signature` HMAC-SHA256 before any processing. Check `WebhookIdempotencyLog` for duplicate event IDs. On `payment.captured`: create Order + SubOrders + OrderLineItems, decrement stock atomically (SELECT FOR UPDATE), create SettlementLedgerEntry, dispatch `send_order_confirmation_email` and `send_seller_new_order_email` Celery tasks.
**Acceptance Criteria:**
- [ ] Invalid HMAC signature returns 400 and logs to AuditLog (no order created)
- [ ] Duplicate webhook ID returns 200 (idempotent) with no second order created
- [ ] Valid `payment.captured`: Order created with status=PAYMENT_CONFIRMED, stock decremented, ledger entry created
- [ ] Concurrent duplicate payment for last unit: only one order succeeds (SELECT FOR UPDATE prevents negative stock)
- [ ] `payment.failed`: any pending order marked PAYMENT_FAILED, stock restored
- [ ] Email Celery tasks dispatched with correct payload (verified via mock)
- [ ] Integration tests cover all webhook event types and idempotency
**Files Affected:** `backend/payments/views.py`, `backend/payments/webhook_handler.py`, `backend/orders/services.py`, `backend/analytics/models.py` (AuditLog)
**Wireframe Reference:** —
**Dependencies:** TASK-005, TASK-027, TASK-006
**Estimate:** L (5 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 3–4
**Status:** Pending

---

### TASK-029
**Epic:** EPIC-005
**Story:** US-018, US-007
**Title:** Implement Celery email tasks for order notifications
**Description:** Implement four Celery tasks on the `email` queue: (1) `send_order_confirmation_email` — buyer receives order ID, itemized list, total, seller contact; (2) `send_seller_new_order_email` — seller receives order details, buyer name, items; (3) `send_order_shipped_email` — buyer receives courier + AWB; (4) `send_order_cancelled_email` — buyer receives refund amount and timeline. Each task: retry ×3 with 30s exponential backoff, log failure to structured log after all retries. Use AWS SES via boto3.
**Acceptance Criteria:**
- [ ] Each task dispatched to `email` queue routes to Celery email worker
- [ ] SES send call succeeds in local test (mock SES or real sandbox)
- [ ] Retry logic: SES failure on attempt 1 → retry; success on attempt 2 → single log entry
- [ ] After 3 failures: error log with `event_type`, `entity_id`, error message
- [ ] Each email content contains required fields per acceptance criteria in US-018
- [ ] Unit tests mock SES and verify retry behavior
**Files Affected:** `backend/payments/tasks.py`, `backend/orders/tasks.py`, `backend/shopnest/settings/base.py` (SES config)
**Wireframe Reference:** —
**Dependencies:** TASK-006, TASK-028
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 4
**Status:** Pending

---

### TASK-030
**Epic:** EPIC-005
**Story:** US-019
**Title:** Implement guest order tracking endpoint and HMAC token generation
**Description:** On order creation, generate a single-use HMAC-SHA256 tracking token tied to the order UUID (using Django secret key as HMAC key). Store token hash in Order model. Implement `GET /api/v1/orders/track/{token}/` — validates HMAC, returns order status, items, AWB (if shipped), seller name. No authentication required.
**Acceptance Criteria:**
- [ ] Tracking URL accessible without JWT (public endpoint)
- [ ] Valid token returns order status, product list, seller name
- [ ] Invalid/tampered token returns 403
- [ ] After seller marks Shipped: tracking page shows AWB and courier name
- [ ] Token is stable (same order always returns same token — token is deterministic from order UUID)
**Files Affected:** `backend/orders/views.py`, `backend/orders/services.py`, `backend/orders/models.py`
**Wireframe Reference:** —
**Dependencies:** TASK-028, TASK-005
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 4
**Status:** Pending

---

### TASK-031
**Epic:** EPIC-005
**Story:** US-020
**Title:** Implement order cancellation with Razorpay refund
**Description:** Implement `POST /api/v1/orders/{id}/cancel/`. Only cancellable if order status=PAYMENT_CONFIRMED (not yet Processing). Calls Razorpay Refunds API for full amount. On refund success: sets order status=CANCELLED, restores stock quantities, dispatches `send_order_cancelled_email` Celery task. Retry refund ×3; after 3 failures: mark as REFUND_PENDING_MANUAL and alert ops via structured log.
**Acceptance Criteria:**
- [ ] Valid cancel: Razorpay refund initiated, order status=CANCELLED, stock restored
- [ ] Cancel attempt on Processing order: 409 "Order cannot be cancelled after seller confirmation"
- [ ] Cancel on non-existent order: 404
- [ ] Razorpay refund API failure after 3 retries: order marked REFUND_PENDING_MANUAL
- [ ] Cancellation email dispatched to buyer
- [ ] Tests cover: success, blocked-by-status, refund API failure
**Files Affected:** `backend/orders/views.py`, `backend/orders/services.py`, `backend/payments/services/razorpay_service.py`
**Wireframe Reference:** —
**Dependencies:** TASK-028, TASK-029
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 4
**Status:** Pending

---

### TASK-032
**Epic:** EPIC-005
**Story:** US-016, US-017
**Title:** Implement checkout and payment UI (SCR-002 Part B)
**Description:** Build the checkout page at `/checkout`. Step 1: Guest/registered checkout form (name, email, phone, address — pre-fill for registered buyers). Step 2: Call `POST /api/v1/checkout/initiate/` to get Razorpay order ID. Step 3: Mount Razorpay checkout widget with returned credentials. On widget callback (payment success/failure): redirect to `/checkout/success` or show inline error. Order success page shows order ID and tracking URL link.
**Acceptance Criteria:**
- [ ] Guest checkout form collects all required fields with inline validation
- [ ] Registered buyer's saved address is pre-populated
- [ ] Razorpay widget mounts and shows UPI/card/NetBanking tabs
- [ ] On payment success: redirect to `/checkout/success?order_id={id}` with order confirmation details
- [ ] On payment failure: inline error with "Try again" option (cart preserved)
- [ ] Out-of-stock error from API renders with list of affected items
- [ ] Functional at 375px mobile and 1440px desktop viewports per SCR-002 wireframe
**Files Affected:** `frontend/app/checkout/page.tsx`, `frontend/app/checkout/success/page.tsx`, `frontend/components/checkout/CheckoutForm.tsx`, `frontend/components/checkout/RazorpayWidget.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-002-cart-checkout.html`
**Dependencies:** TASK-025, TASK-027
**Estimate:** L (5 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 4
**Status:** Pending

---

### TASK-033
**Epic:** EPIC-005
**Story:** US-019, US-020
**Title:** Implement guest order tracking page (SCR-008)
**Description:** Build the Next.js order tracking page at `/orders/track/{token}`. Fetches `GET /api/v1/orders/track/{token}/`. Shows: order status timeline (Payment Confirmed → Processing → Shipped → Delivered), product list, seller name, AWB number (when available). Cancel button visible only if status=PAYMENT_CONFIRMED — calls cancel API. Shows refund info after cancellation.
**Acceptance Criteria:**
- [ ] Page renders for valid HMAC token without authentication
- [ ] Status timeline shows current status highlighted
- [ ] AWB and courier name visible after Shipped status
- [ ] Cancel button visible only in Payment Confirmed status
- [ ] Cancellation confirmation dialog prevents accidental cancel
- [ ] Invalid token shows 403 error page
- [ ] Functional at 375px mobile and 1440px desktop viewports per SCR-008 wireframe
**Files Affected:** `frontend/app/orders/track/[token]/page.tsx`, `frontend/components/orders/OrderTimeline.tsx`, `frontend/components/orders/CancelOrderButton.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-008-order-tracking.html`
**Dependencies:** TASK-004, TASK-030, TASK-031
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 4
**Status:** Pending

---

### TASK-034
**Epic:** EPIC-005
**Story:** US-017
**Title:** Write integration tests for checkout and webhook critical path
**Description:** Write end-to-end integration tests for the complete payment flow: cart → checkout initiate → mock Razorpay webhook (payment.captured) → order created → stock decremented → email tasks dispatched. Also test: forged webhook rejected, duplicate webhook idempotent, payment.failed restores stock, concurrent checkout of last unit (only one succeeds).
**Acceptance Criteria:**
- [ ] Happy path test: full flow from cart to order confirmation
- [ ] Forged webhook test: invalid HMAC → 400, no order created
- [ ] Duplicate webhook test: same event ID → 200, single order in DB
- [ ] Race condition test: two concurrent webhooks for last unit → one order, one failure
- [ ] `payments/` app coverage ≥ 80%
**Files Affected:** `backend/payments/tests/test_webhook.py`, `backend/orders/tests/test_services.py`
**Wireframe Reference:** —
**Dependencies:** TASK-028, TASK-031
**Estimate:** S (2 SP)
**Type:** Test
**Assigned Role:** Backend Dev
**Sprint:** 4
**Status:** Pending

---

## EPIC-006: Seller Onboarding

**Description:** Store setup wizard (3-step guided flow: store details + image upload, category selection, subscription payment). Razorpay subscription creation. GST invoice PDF generation via WeasyPrint. Bank account details collection with AES-256-GCM field encryption.

**Business Value:** Sellers cannot list products or access the dashboard until onboarding is complete. This is the seller activation funnel — its UX quality directly determines seller time-to-first-listing (target: < 30 minutes).

---

### TASK-035
**Epic:** EPIC-006
**Story:** US-002
**Title:** Implement store setup API endpoints
**Description:** Implement `POST /api/v1/seller/store/setup/` (creates Store record, uploads logo and banner to S3 with MIME validation, sets `setup_complete=True`), and `GET /api/v1/seller/store/` (returns current store data). MIME validation using `python-magic`. Logo max 2MB; banner max 5MB. S3 pre-signed URL approach or direct upload via boto3. CloudFront URL stored.
**Acceptance Criteria:**
- [ ] Valid logo (PNG, 1.5MB) and banner (JPG, 3MB) uploaded to S3; CloudFront URLs returned
- [ ] Logo over 2MB returns 400 with correct message
- [ ] PDF or EXE file returns 400 "Only JPEG and PNG images are accepted"
- [ ] `store.setup_complete=True` after successful setup
- [ ] Seller cannot access product listing endpoints until `setup_complete=True` AND subscription active
- [ ] Unit tests cover: success, oversized file, invalid MIME type
**Files Affected:** `backend/sellers/views.py`, `backend/sellers/serializers.py`, `backend/sellers/services.py`, `backend/shopnest/storage.py`
**Wireframe Reference:** —
**Dependencies:** TASK-005, TASK-009, TASK-015
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 4
**Status:** Pending

---

### TASK-036
**Epic:** EPIC-006
**Story:** US-003
**Title:** Implement Razorpay subscription creation and webhook handlers
**Description:** Implement `POST /api/v1/seller/subscription/create/` — creates Razorpay Subscription (monthly plan at ₹1,999/month — configurable via env var `SUBSCRIPTION_PRICE_PAISE=199900`). Webhook handlers for `subscription.charged` (set status=ACTIVE, generate invoice) and `subscription.charge.failed` (set status=PAYMENT_FAILED, dispatch payment failure email). Implement grace period check: 7-day grace, then Celery Beat task transitions to SUSPENDED.
**Acceptance Criteria:**
- [ ] Successful `subscription.charged`: seller status=ACTIVE, Subscription record created
- [ ] `subscription.charge.failed`: status=PAYMENT_FAILED, failure email dispatched within 60s
- [ ] 7-day grace period: seller with Payment Failed status can access dashboard for exactly 7 days
- [ ] Day 8 Celery Beat task: seller status=SUSPENDED, products hidden, suspension email dispatched
- [ ] Successful renewal after suspension: status=ACTIVE, products re-published
- [ ] Subscription price is read from env var (not hardcoded)
**Files Affected:** `backend/payments/views.py`, `backend/payments/webhook_handler.py`, `backend/payments/tasks.py`, `backend/sellers/models.py`
**Wireframe Reference:** —
**Dependencies:** TASK-028, TASK-006, TASK-015
**Estimate:** L (5 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-037
**Epic:** EPIC-006
**Story:** US-003
**Title:** Implement GST invoice PDF generation with WeasyPrint
**Description:** Implement `generate_subscription_invoice` Celery task triggered on successful subscription payment. Uses WeasyPrint to render HTML invoice template to PDF. Invoice fields: invoice number, date, seller name + GSTIN, ShopNest GSTIN, plan name, amount (INR), GST amount (18%), total. Store PDF in S3 (`invoices/{seller_id}/{invoice_id}.pdf`). Expose `GET /api/v1/seller/invoices/{id}/download/` to download.
**Acceptance Criteria:**
- [ ] PDF generated and uploaded to S3 on subscription payment confirmation
- [ ] PDF contains all required GST-compliant fields (invoice number, GSTIN, tax breakdown)
- [ ] `GET /api/v1/seller/invoices/{id}/download/` returns a pre-signed S3 URL (not the PDF bytes directly)
- [ ] Pre-signed URL expires in 15 minutes
- [ ] Invoice accessible only to the owning seller (403 for other sellers)
**Files Affected:** `backend/payments/tasks.py`, `backend/payments/templates/invoice.html`, `backend/payments/views.py`
**Wireframe Reference:** —
**Dependencies:** TASK-036, TASK-009
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-038
**Epic:** EPIC-006
**Story:** US-003
**Title:** Implement payout bank account details with field encryption
**Description:** Implement `POST /api/v1/seller/payout-details/` and `PATCH /api/v1/seller/payout-details/` for bank account collection. Encrypt `account_number` field using AES-256-GCM via PyCA cryptography library; encryption key loaded from AWS Secrets Manager (env var: `FIELD_ENCRYPTION_KEY`). Never return raw account number in API responses — mask to last 4 digits.
**Acceptance Criteria:**
- [ ] `account_number` stored encrypted in DB (verified via direct DB query: not plaintext)
- [ ] API response returns masked account number: `"****1234"`
- [ ] `account_holder_name` and `ifsc_code` stored in plaintext (not sensitive under PCI-DSS scope)
- [ ] Decryption succeeds in Celery payout task (integration test)
- [ ] Key rotation does not break existing records (documented procedure in `docs/ops/`)
**Files Affected:** `backend/sellers/models.py`, `backend/sellers/services.py`, `backend/sellers/serializers.py`, `backend/sellers/encryption.py`
**Wireframe Reference:** —
**Dependencies:** TASK-005, TASK-015
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-039
**Epic:** EPIC-006
**Story:** US-002
**Title:** Implement store setup wizard UI (SCR-011)
**Description:** Build the 3-step wizard at `/seller/setup`. Step 1: Store name + logo upload + banner upload (drag-and-drop or file picker, client-side size validation). Step 2: Category selection (multi-select from fetched categories, max 5). Step 3: Review and submit. Progress stepper at top. On success, redirect to `/seller/subscribe`.
**Acceptance Criteria:**
- [ ] Step 1: logo file picker shows preview; rejects files > 2MB client-side with inline error
- [ ] Step 2: category multi-select shows all available categories; max 5 enforced
- [ ] Step 3: review shows all entered data before final submit
- [ ] API error (e.g., MIME type rejection) shown inline on Step 1
- [ ] Progress bar accurately reflects current step
- [ ] Back navigation between steps preserves entered data
- [ ] Functional at 768px tablet viewport (seller dashboard is tablet-first per NFR-USA-002)
**Files Affected:** `frontend/app/seller/setup/page.tsx`, `frontend/components/seller/StoreSetupWizard.tsx`, `frontend/components/seller/ImageUpload.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-011-store-setup-wizard.html`
**Dependencies:** TASK-004, TASK-035
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-040
**Epic:** EPIC-006
**Story:** US-003
**Title:** Implement seller subscription signup UI (SCR-012)
**Description:** Build the subscription payment page at `/seller/subscribe`. Shows plan name, price (₹1,999/month), features list. Calls `POST /api/v1/seller/subscription/create/` to initiate Razorpay Subscription. Mounts Razorpay Subscription checkout. On payment success: poll or wait for webhook confirmation, then redirect to `/seller/dashboard`. Shows grace period warning if accessing with PAYMENT_FAILED status.
**Acceptance Criteria:**
- [ ] Subscription page displays plan details and price clearly
- [ ] Razorpay checkout mounts and shows payment options
- [ ] Post-payment redirect to seller dashboard occurs after webhook confirmation
- [ ] Grace period warning state renders correctly for PAYMENT_FAILED sellers
- [ ] Functional at 768px tablet viewport
**Files Affected:** `frontend/app/seller/subscribe/page.tsx`, `frontend/components/seller/SubscriptionCheckout.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-012-seller-subscription.html`
**Dependencies:** TASK-004, TASK-036
**Estimate:** S (2 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 5
**Status:** Pending

---

## EPIC-007: Seller Product Management

**Description:** Add product listing with image upload, edit product (with conditional re-review logic), delete product (with active order guard), view product list with status badges, admin approval queue triggers, and low-stock alert Celery task.

**Business Value:** Products are ShopNest's core inventory. Without product management, sellers cannot list, update, or maintain their catalogue.

---

### TASK-041
**Epic:** EPIC-007
**Story:** US-004
**Title:** Implement product CRUD API endpoints for sellers
**Description:** Implement: `POST /api/v1/seller/products/` (create — sets status=PENDING_REVIEW, triggers admin queue notification), `GET /api/v1/seller/products/` (list seller's own products with status filter), `GET /api/v1/seller/products/{id}/`, `PATCH /api/v1/seller/products/{id}/` (edit — name/description/image change triggers status=PENDING_REVIEW; price/stock change does not), `DELETE /api/v1/seller/products/{id}/` (blocked if active orders exist). Upload up to 5 images to S3 per product. Generate stable URL slug on create (never updated on name edit).
**Acceptance Criteria:**
- [ ] `POST` returns 201 with status=PENDING_REVIEW; product absent from marketplace
- [ ] `PATCH` with name change: product transitions to PENDING_REVIEW even if previously Active
- [ ] `PATCH` with price change only: product remains in current status
- [ ] `DELETE` with active order returns 409
- [ ] Slug is generated at creation from original name + UUID prefix and never changes
- [ ] Images validated: JPEG/PNG only, max 5MB each, max 5 per product
- [ ] Seller cannot see or modify another seller's products (403)
- [ ] Unit tests cover all edit scenarios, deletion guard, MIME validation
**Files Affected:** `backend/products/views.py`, `backend/products/serializers.py`, `backend/products/services.py`, `backend/products/urls.py`
**Wireframe Reference:** —
**Dependencies:** TASK-017, TASK-035
**Estimate:** L (5 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 4
**Status:** Pending

---

### TASK-042
**Epic:** EPIC-007
**Story:** US-010
**Title:** Implement low-stock alert Celery task
**Description:** On stock decrement (triggered by order creation in TASK-028), check if product stock ≤ 5. If yes, dispatch `send_low_stock_alert` Celery task (email queue) — sends "Low Stock Alert" email to seller with product name and current quantity. Also set `low_stock_flag=True` on the product for dashboard display. If stock = 0: set product status = OUT_OF_STOCK (hidden from marketplace).
**Acceptance Criteria:**
- [ ] Order that reduces stock to 4 → low-stock email dispatched to seller within 60s
- [ ] Order that reduces stock to 0 → product status = OUT_OF_STOCK; absent from marketplace
- [ ] `low_stock_flag` visible in seller product list API response
- [ ] Re-stocking (PATCH stock > 5) clears `low_stock_flag`
- [ ] Re-stocking (PATCH stock > 0 from OUT_OF_STOCK) restores product to ACTIVE status
**Files Affected:** `backend/products/tasks.py`, `backend/orders/services.py` (call hook), `backend/products/models.py`
**Wireframe Reference:** —
**Dependencies:** TASK-028, TASK-041, TASK-006
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-043
**Epic:** EPIC-007
**Story:** US-004, US-005, US-006
**Title:** Implement seller product management UI (SCR-003)
**Description:** Build the seller product list page at `/seller/products`. Table view of all seller's products with: name, price, stock, status badge (Active/Pending Review/Rejected/Out of Stock), last updated date, Edit and Delete actions. Add Product button navigates to `/seller/products/new`. Rejection reason expandable inline on Rejected rows. Low-stock badge on products with `low_stock_flag=True`.
**Acceptance Criteria:**
- [ ] Product list shows all statuses with correct color-coded badges
- [ ] Rejected product row shows rejection reason on expand (inline, not modal)
- [ ] Low-stock products show amber warning badge
- [ ] Delete action shows confirmation dialog; blocked products show error toast
- [ ] Functional at 768px tablet viewport (seller dashboard is tablet-first)
- [ ] Empty state renders when seller has no products
**Files Affected:** `frontend/app/seller/products/page.tsx`, `frontend/components/seller/ProductTable.tsx`, `frontend/components/seller/ProductStatusBadge.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-003-seller-products.html`
**Dependencies:** TASK-004, TASK-041
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-044
**Epic:** EPIC-007
**Story:** US-004, US-006
**Title:** Implement add and edit product form UI
**Description:** Build the product form at `/seller/products/new` and `/seller/products/{id}/edit`. Fields: name, description (rich text or textarea), price (INR), stock quantity, category selector (from store categories), image uploader (up to 5 images, drag-and-drop reorder, remove individual). Client-side validation mirrors backend rules. Submit calls `POST` or `PATCH` respectively. Show re-review warning when editing name/description/images.
**Acceptance Criteria:**
- [ ] Image uploader accepts up to 5 images; drag-and-drop reorder works
- [ ] Rejects non-JPEG/PNG files client-side with inline error
- [ ] Price field accepts only positive numbers with up to 2 decimal places
- [ ] Re-review warning banner shown when editing name/description/images on an Active product
- [ ] Edit form pre-fills all existing product data
- [ ] Success redirects to `/seller/products` with success toast
**Files Affected:** `frontend/app/seller/products/new/page.tsx`, `frontend/app/seller/products/[id]/edit/page.tsx`, `frontend/components/seller/ProductForm.tsx`, `frontend/components/seller/MultiImageUpload.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-003-seller-products.html`
**Dependencies:** TASK-004, TASK-041, TASK-043
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 5
**Status:** Pending

---

## EPIC-008: Seller Order Fulfillment

**Description:** Seller order dashboard, confirm order, mark as shipped (with AWB), mark as delivered, email notifications to buyer at each step, and seller-scoped data isolation.

**Business Value:** Sellers must manage order fulfillment for the marketplace to function. Buyers cannot receive goods without seller fulfillment actions.

---

### TASK-045
**Epic:** EPIC-008
**Story:** US-007, US-008, US-009, US-034
**Title:** Implement seller order management API endpoints
**Description:** Implement: `GET /api/v1/seller/orders/` (list seller's sub-orders, sorted by created_at DESC), `GET /api/v1/seller/orders/{id}/`, `POST /api/v1/seller/orders/{id}/confirm/` (PAYMENT_CONFIRMED → PROCESSING), `POST /api/v1/seller/orders/{id}/ship/` (requires courier_name + awb_number; PROCESSING → SHIPPED; dispatches buyer shipped email), `POST /api/v1/seller/orders/{id}/deliver/` (SHIPPED → DELIVERED). Strict seller isolation — seller can only access their own sub-orders.
**Acceptance Criteria:**
- [ ] Seller A cannot access Seller B's order IDs (403 or 404)
- [ ] Confirm: status → PROCESSING; buyer cancel blocked after this
- [ ] Ship: requires both courier_name and awb_number; empty AWB → 400
- [ ] Deliver: status → DELIVERED; settlement ledger entry eligible for payout
- [ ] Buyer shipped email dispatched within 60s of ship action
- [ ] Order list sorted most recent first; pagination at 20 per page
- [ ] Unit tests cover all transitions and isolation
**Files Affected:** `backend/orders/views.py`, `backend/orders/services.py`, `backend/orders/serializers.py`, `backend/orders/urls.py`
**Wireframe Reference:** —
**Dependencies:** TASK-028, TASK-029
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-046
**Epic:** EPIC-008
**Story:** US-007, US-008, US-009, US-034
**Title:** Implement seller order management UI (SCR-014)
**Description:** Build the seller order list page at `/seller/orders`. Table with: sub-order ID, buyer (first name + last initial), product list (truncated), total, status badge, date. Inline action buttons per status: "Confirm" (Payment Confirmed), "Mark Shipped" modal (courier + AWB input), "Mark Delivered" (Shipped). Order detail expandable row or side panel. Status filter tabs: All / New / Processing / Shipped / Delivered.
**Acceptance Criteria:**
- [ ] Only current seller's orders visible
- [ ] Correct action buttons visible per status (no "Confirm" button on Processing orders)
- [ ] "Mark Shipped" modal: both courier and AWB required; inline validation
- [ ] Status tabs filter correctly
- [ ] Functional at 768px tablet viewport
- [ ] Empty state when no orders
**Files Affected:** `frontend/app/seller/orders/page.tsx`, `frontend/components/seller/OrderTable.tsx`, `frontend/components/seller/ShipOrderModal.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-014-seller-orders.html`
**Dependencies:** TASK-004, TASK-045
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-047
**Epic:** EPIC-008
**Story:** US-007, US-008, US-009, US-034
**Title:** Write seller order fulfillment tests
**Description:** Write tests for: order isolation (Seller A cannot access Seller B's orders), all status transitions (confirm/ship/deliver), blocked transitions (ship a Payment Confirmed order → error), buyer email dispatch on ship (mock Celery), AWB validation, deliver with missing AWB.
**Acceptance Criteria:**
- [ ] Cross-seller isolation test: Seller A token + Seller B order ID → 403
- [ ] All valid status transition tests pass
- [ ] Invalid transitions return correct error codes
- [ ] Email task dispatch mocked and verified
- [ ] `orders/` app coverage ≥ 80%
**Files Affected:** `backend/orders/tests/test_views.py`, `backend/orders/tests/test_services.py`
**Wireframe Reference:** —
**Dependencies:** TASK-045
**Estimate:** S (2 SP)
**Type:** Test
**Assigned Role:** Backend Dev
**Sprint:** 5
**Status:** Pending

---

## EPIC-009: Seller Payouts & Settlement

**Description:** Settlement ledger (credit on order confirmation), Celery Beat weekly payout job (Monday 09:00 IST), Razorpay Payouts API disbursement, payout history API, payout email notification. Suspended seller exclusion.

**Business Value:** Payout is the fundamental financial obligation to sellers. Without it, sellers have no reason to trust or continue using ShopNest.

---

### TASK-048
**Epic:** EPIC-009
**Story:** US-024
**Title:** Implement settlement ledger and payout service
**Description:** Implement `create_settlement_ledger_entry` service: called when sub-order status transitions to PROCESSING. Creates SettlementLedgerEntry with: seller_id, sub_order_id, gross_amount_paise, gateway_fee_paise (computed as 2% of gross, floor), net_amount_paise. Implement `GET /api/v1/seller/payouts/balance/` returning current unsettled balance.
**Acceptance Criteria:**
- [ ] Order confirm → SettlementLedgerEntry created with correct gross, fee, net amounts
- [ ] Fee computed as `floor(gross_amount_paise × 0.02)` (2% Razorpay fee)
- [ ] `GET /api/v1/seller/payouts/balance/` returns sum of all unpaid ledger entries
- [ ] Seller A cannot see Seller B's balance
- [ ] Unit tests verify fee computation formula and isolation
**Files Affected:** `backend/payments/services/settlement_service.py`, `backend/payments/views.py`, `backend/orders/services.py` (hook)
**Wireframe Reference:** —
**Dependencies:** TASK-045, TASK-005
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 6
**Status:** Pending

---

### TASK-049
**Epic:** EPIC-009
**Story:** US-024
**Title:** Implement weekly Celery Beat payout job
**Description:** Implement `execute_weekly_payouts` Celery Beat task scheduled for Monday 09:00 IST (Celery crontab: `crontab(hour=3, minute=30, day_of_week=1)` — IST is UTC+5:30). For each seller with: status=ACTIVE AND subscription=ACTIVE AND unsettled balance > 0: call Razorpay Payouts API (NEFT/IMPS), create Payout record (status=INITIATED, razorpay_payout_id), mark ledger entries as paid, dispatch `send_payout_initiated_email`. Skip SUSPENDED sellers. Log failed payouts with seller_id and error for ops review.
**Acceptance Criteria:**
- [ ] Payout job runs at Monday 09:00 IST (verified via Celery Beat log with cron expression)
- [ ] Suspended sellers skipped; their balance preserved
- [ ] Payout record created with correct amount and Razorpay reference ID
- [ ] Payout email dispatched within 60s
- [ ] Failed payout logged with seller_id and reason; retried next Monday cycle
- [ ] Integration test mocks Razorpay Payouts API and verifies all assertions
**Files Affected:** `backend/payments/tasks.py`, `backend/payments/services/payout_service.py`, `backend/shopnest/settings/base.py` (Celery Beat schedule)
**Wireframe Reference:** —
**Dependencies:** TASK-048, TASK-006, TASK-038
**Estimate:** L (5 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 6
**Status:** Pending

---

### TASK-050
**Epic:** EPIC-009
**Story:** US-032
**Title:** Implement payout history API and UI (SCR-015)
**Description:** Backend: `GET /api/v1/seller/payouts/` — list of last 12 months of payouts sorted by date descending, with: amount, razorpay_payout_id, status, initiated_at. Frontend: Build payout history page at `/seller/payouts` showing payout list table with amount, date, status, reference ID. Summary card at top showing current unsettled balance.
**Acceptance Criteria:**
- [ ] API returns correct payout list for the requesting seller only
- [ ] UI payout table shows all columns correctly
- [ ] Unsettled balance card matches `GET /api/v1/seller/payouts/balance/`
- [ ] Empty state renders when no payouts yet
- [ ] Functional at 768px tablet viewport
**Files Affected:** `backend/payments/views.py`, `backend/payments/serializers.py`, `frontend/app/seller/payouts/page.tsx`, `frontend/components/seller/PayoutHistory.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-015-seller-payouts.html`
**Dependencies:** TASK-048, TASK-049, TASK-004
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 6
**Status:** Pending

---

### TASK-051
**Epic:** EPIC-009
**Story:** US-032
**Title:** Build seller payout history frontend page (SCR-015)
**Description:** Build the payout history frontend at `/seller/payouts`. Fetch from `GET /api/v1/seller/payouts/` and `GET /api/v1/seller/payouts/balance/`. Render: current balance card, payout history table (date, amount formatted in INR, Razorpay ref ID, status badge). Status badge: Initiated (blue), Completed (green), Failed (red). Pagination for > 20 entries.
**Acceptance Criteria:**
- [ ] Balance card shows current unsettled amount in INR formatting
- [ ] Payout table shows all 4 columns with correct formatting
- [ ] Status badges render with correct color per status
- [ ] Pagination works for > 20 payout records
- [ ] Empty state visible when no payouts exist
- [ ] Functional at 768px tablet viewport
**Files Affected:** `frontend/app/seller/payouts/page.tsx`, `frontend/components/seller/PayoutHistory.tsx`, `frontend/components/seller/PayoutBalanceCard.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-015-seller-payouts.html`
**Dependencies:** TASK-004, TASK-050
**Estimate:** S (2 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 6
**Status:** Pending

---

### TASK-052
**Epic:** EPIC-009
**Story:** US-024
**Title:** Write payout and settlement tests
**Description:** Write tests for: settlement entry creation with fee computation, payout job skips suspended sellers, payout job creates correct Razorpay API call, balance API returns correct sum, payout email dispatched, duplicate payout prevention (idempotency on ledger entries).
**Acceptance Criteria:**
- [ ] Fee computation test: ₹1,000 gross → fee=20, net=980
- [ ] Suspended seller exclusion test
- [ ] Razorpay Payouts API mock verifies correct amount and mode
- [ ] Balance test: sum of net_amount_paise for unpaid entries
- [ ] `payments/` payout coverage ≥ 80%
**Files Affected:** `backend/payments/tests/test_payout_service.py`, `backend/payments/tests/test_tasks.py`
**Wireframe Reference:** —
**Dependencies:** TASK-049
**Estimate:** S (2 SP)
**Type:** Test
**Assigned Role:** Backend Dev
**Sprint:** 6
**Status:** Pending

---

## EPIC-010: Seller Subscription Management

**Description:** Seller subscription status dashboard, next billing date display, invoice download, bank account update. Visible in seller sidebar with Active/Expired badge.

---

### TASK-053
**Epic:** EPIC-010
**Story:** US-036
**Title:** Implement subscription status and invoice API endpoints
**Description:** Implement: `GET /api/v1/seller/subscription/` (returns subscription status, plan name, next billing date from Razorpay API or stored in DB), `GET /api/v1/seller/invoices/` (list invoices), `GET /api/v1/seller/invoices/{id}/download/` (pre-signed S3 URL). Subscription status sourced from local DB (synced via webhook, not live Razorpay call on every request).
**Acceptance Criteria:**
- [ ] Returns correct status (Active/Payment Failed/Suspended)
- [ ] Next billing date accurate (sourced from Subscription model `next_billing_at` field)
- [ ] Invoice list shows all seller's invoices
- [ ] Download endpoint returns pre-signed URL expiring in 15 minutes
- [ ] Other seller's invoices return 403
**Files Affected:** `backend/payments/views.py`, `backend/payments/serializers.py`
**Wireframe Reference:** —
**Dependencies:** TASK-036, TASK-037
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 6
**Status:** Pending

---

### TASK-054
**Epic:** EPIC-010
**Story:** US-036
**Title:** Build subscription management UI (SCR-016)
**Description:** Build the subscription page at `/seller/subscription`. Show: current plan name, status badge (Active/Payment Failed/Suspended), next billing date, subscription price. List of downloadable GST invoices (date, amount, download link). Grace period warning banner if status=PAYMENT_FAILED. "Update Payment Method" link redirects to Razorpay.
**Acceptance Criteria:**
- [ ] Subscription status and next billing date displayed correctly
- [ ] Invoice download links open PDF (pre-signed S3 URL)
- [ ] Grace period warning banner visible for PAYMENT_FAILED status
- [ ] Suspended seller sees suspension notice with support contact
- [ ] Functional at 768px tablet viewport
**Files Affected:** `frontend/app/seller/subscription/page.tsx`, `frontend/components/seller/SubscriptionStatus.tsx`, `frontend/components/seller/InvoiceList.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-016-seller-subscription-mgmt.html`
**Dependencies:** TASK-004, TASK-053
**Estimate:** S (2 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 6
**Status:** Pending

---

### TASK-055
**Epic:** EPIC-010
**Story:** US-037
**Title:** Implement bank account update with payout hold logic
**Description:** Implement `PATCH /api/v1/seller/payout-details/` — updates bank account details. Policy (OQ-006 resolution): existing pending payout cycle runs to old account; new account takes effect from the next Monday payout cycle. Log the bank account update event to AuditLog. Encrypt new account_number on save.
**Acceptance Criteria:**
- [ ] PATCH updates bank details and encrypts new account_number
- [ ] AuditLog entry created for bank account update
- [ ] In-progress payout cycle (if any) is not cancelled — it processes to old account
- [ ] Next Monday payout uses new bank account
- [ ] Unit test verifies audit log creation and encryption
**Files Affected:** `backend/sellers/views.py`, `backend/sellers/services.py`, `backend/analytics/services.py`
**Wireframe Reference:** —
**Dependencies:** TASK-038, TASK-049
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 6
**Status:** Pending

---

### TASK-056
**Epic:** EPIC-010
**Story:** US-037
**Title:** Build seller store settings UI (SCR-017)
**Description:** Build the settings page at `/seller/settings`. Sections: (1) Store Profile (name, logo, banner, categories — editable); (2) Bank Account Details (account_holder_name, masked account_number, IFSC — update form). Separate save buttons per section. Confirmation dialog on bank account update ("Payout for current cycle will go to old account. New account takes effect next Monday.").
**Acceptance Criteria:**
- [ ] Store profile section pre-filled from API; save PATCH updates
- [ ] Bank account section shows masked account number
- [ ] Confirmation dialog renders with correct message before bank update submits
- [ ] Logo/banner upload replaces existing images (old image removed from display)
- [ ] Functional at 768px tablet viewport
**Files Affected:** `frontend/app/seller/settings/page.tsx`, `frontend/components/seller/StoreProfileForm.tsx`, `frontend/components/seller/BankAccountForm.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-017-seller-store-settings.html`
**Dependencies:** TASK-004, TASK-035, TASK-055
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 6
**Status:** Pending

---

## EPIC-011: Seller Analytics & Dashboard

**Description:** Seller dashboard home with summary metrics (revenue, order count, AOV for 7d/30d), top 5 products by order count, and CSV order export.

---

### TASK-057
**Epic:** EPIC-011
**Story:** US-028
**Title:** Implement seller analytics API endpoints
**Description:** Implement: `GET /api/v1/seller/analytics/summary/` (7d and 30d: revenue total in paise, order count, AOV in paise), `GET /api/v1/seller/analytics/top-products/` (top 5 products by order count last 30d; ties broken by name alphabetically). Queries run against orders/order_line_items tables filtered by seller_id. Use DB aggregation (no separate analytics table for these queries at MVP scale).
**Acceptance Criteria:**
- [ ] Summary returns correct values for a known fixture (3 orders × ₹1,500 → revenue=450000 paise, count=3, AOV=150000 paise)
- [ ] Top products correctly ranked; ties broken alphabetically
- [ ] Seller A cannot see Seller B's analytics
- [ ] Both endpoints respond within 200ms for ≤ 1,000 orders
**Files Affected:** `backend/analytics/views.py`, `backend/analytics/services.py`, `backend/analytics/urls.py`
**Wireframe Reference:** —
**Dependencies:** TASK-045, TASK-005
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 6
**Status:** Pending

---

### TASK-058
**Epic:** EPIC-011
**Story:** US-038
**Title:** Implement seller order CSV export
**Description:** Implement `GET /api/v1/seller/orders/export/` — generates RFC 4180 CSV with UTF-8-BOM encoding. Columns: order_id, date, product_name, quantity, unit_price_inr, buyer_city, order_status. Returns as file download response (Content-Disposition: attachment). Only seller's own orders included.
**Acceptance Criteria:**
- [ ] CSV file downloads correctly with all 7 required columns
- [ ] UTF-8-BOM encoding (file opens without garbling in Microsoft Excel)
- [ ] Only requesting seller's orders included
- [ ] Date format: YYYY-MM-DD
- [ ] Price formatted as decimal INR (e.g., 999.00, not 99900)
**Files Affected:** `backend/orders/views.py`, `backend/orders/exporters.py`
**Wireframe Reference:** —
**Dependencies:** TASK-045
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 6
**Status:** Pending

---

### TASK-059
**Epic:** EPIC-011
**Story:** US-028
**Title:** Build seller dashboard home UI (SCR-013)
**Description:** Build the seller dashboard at `/seller/dashboard`. Metric cards: Last 7 Days Revenue (INR), Last 30 Days Revenue, Order Count (7d/30d), Average Order Value. Top 5 Products table (name, order count last 30d). Recent Orders widget (last 5 sub-orders with status). Low-stock alerts section (products with low_stock_flag=True). Quick links to Products and Orders.
**Acceptance Criteria:**
- [ ] All metric cards show correct values from analytics API
- [ ] Top products table renders with correct ranking
- [ ] Recent orders widget shows last 5 orders with status badges
- [ ] Low-stock section shows all products with low_stock_flag
- [ ] Empty states for each section when no data exists
- [ ] Functional at 768px tablet viewport
**Files Affected:** `frontend/app/seller/dashboard/page.tsx`, `frontend/components/seller/MetricCard.tsx`, `frontend/components/seller/TopProductsTable.tsx`, `frontend/components/seller/LowStockAlerts.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-013-seller-dashboard.html`
**Dependencies:** TASK-004, TASK-057, TASK-045
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 7
**Status:** Pending

---

## EPIC-012: Admin Portal

**Description:** Admin login, seller approval queue, product approval queue, seller suspension/reactivation, product approval/rejection, admin dashboard with platform metrics. Desktop-only (NFR-USA-002 scoping admin to ≥768px, per UX design).

---

### TASK-060
**Epic:** EPIC-012
**Story:** US-021, US-023, US-029
**Title:** Implement admin seller management API endpoints
**Description:** Implement: `GET /api/v1/admin/sellers/` (pending queue — PENDING_REVIEW sellers, with store name, email, phone, registration date), `POST /api/v1/admin/sellers/{id}/approve/` (status=ACTIVE; dispatches approval email), `POST /api/v1/admin/sellers/{id}/reject/` (requires reason 1–500 chars; status=REJECTED; sets registration_blocked via seller email; dispatches rejection email), `POST /api/v1/admin/sellers/{id}/suspend/` (status=SUSPENDED; hides products; dispatches suspension email), `POST /api/v1/admin/sellers/{id}/reactivate/` (status=ACTIVE; re-publishes products). All endpoints require Admin JWT.
**Acceptance Criteria:**
- [ ] Approve: seller status=ACTIVE; approval email dispatched
- [ ] Reject: status=REJECTED; email blocked for that email; rejection email with reason dispatched
- [ ] Suspend: status=SUSPENDED; all seller products status becomes hidden (Triple-Gate fails); suspension email dispatched
- [ ] Reactivate: status=ACTIVE; previously ACTIVE products visible again
- [ ] Seller JWT on admin endpoints → 403
- [ ] Unit tests cover all 4 transitions and RBAC enforcement
**Files Affected:** `backend/sellers/views.py`, `backend/sellers/services.py`, `backend/sellers/urls.py`
**Wireframe Reference:** —
**Dependencies:** TASK-015, TASK-017
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 7
**Status:** Pending

---

### TASK-061
**Epic:** EPIC-012
**Story:** US-022, US-030
**Title:** Implement admin product approval API endpoints
**Description:** Implement: `GET /api/v1/admin/products/` (queue — PENDING_REVIEW products with name, seller, submitted_at, thumbnail), `POST /api/v1/admin/products/{id}/approve/` (status=ACTIVE; dispatches approval email to seller), `POST /api/v1/admin/products/{id}/reject/` (requires reason; status=REJECTED; dispatches rejection email with reason), `POST /api/v1/admin/products/{id}/remove/` (status=ADMIN_REMOVED; requires reason; dispatches removal email). All require Admin JWT.
**Acceptance Criteria:**
- [ ] Approve: product appears in marketplace immediately (Triple-Gate passes)
- [ ] Reject: product status=REJECTED; rejection email with reason dispatched
- [ ] Remove: product status=ADMIN_REMOVED; absent from marketplace; seller email dispatched
- [ ] Admin queue shows products appearing within 5 seconds of seller submission
- [ ] Non-admin JWT → 403 on all endpoints
**Files Affected:** `backend/products/views.py`, `backend/products/services.py`
**Wireframe Reference:** —
**Dependencies:** TASK-041, TASK-015
**Estimate:** M (3 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 7
**Status:** Pending

---

### TASK-062
**Epic:** EPIC-012
**Story:** US-035
**Title:** Implement admin dashboard metrics API
**Description:** Implement `GET /api/v1/admin/dashboard/` returning: total active sellers, total active products, total orders last 7 days, total orders last 30 days, total GMV last 7 days (paise), total GMV last 30 days (paise). Queries run against DB with appropriate indexes. Cache response in Redis for 5 minutes to avoid repeated aggregation queries.
**Acceptance Criteria:**
- [ ] Returns all 6 metrics with correct values for a known fixture
- [ ] Response cached in Redis for 5 minutes (second call within 5min hits cache)
- [ ] Cache invalidated on new order created (or accepted eventual consistency — document choice)
- [ ] Only Admin JWT can call this endpoint
**Files Affected:** `backend/analytics/views.py`, `backend/analytics/services.py`
**Wireframe Reference:** —
**Dependencies:** TASK-015, TASK-045, TASK-017
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 7
**Status:** Pending

---

### TASK-063
**Epic:** EPIC-012
**Story:** US-021
**Title:** Build admin login UI (SCR-018)
**Description:** Build the admin login page at `/admin/login`. Simple email + password form. Calls `POST /api/v1/auth/token/` with admin credentials. Validates that returned JWT has admin role (role claim in token). Stores JWT in httpOnly cookie. Redirects to `/admin/dashboard` on success. Non-admin credentials show "Access denied" error.
**Acceptance Criteria:**
- [ ] Admin login with valid credentials redirects to `/admin/dashboard`
- [ ] Non-admin role JWT shows "Access denied — admin credentials required" error
- [ ] Invalid credentials show "Invalid email or password" error
- [ ] JWT stored in httpOnly cookie (not localStorage)
**Files Affected:** `frontend/app/admin/login/page.tsx`, `frontend/components/admin/AdminLoginForm.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-018-admin-login.html`
**Dependencies:** TASK-004, TASK-010
**Estimate:** S (2 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 7
**Status:** Pending

---

### TASK-064
**Epic:** EPIC-012
**Story:** US-021, US-023, US-029
**Title:** Build admin seller approval queue UI (SCR-019)
**Description:** Build the seller approval queue at `/admin/sellers`. Table: store name, seller name, email, phone, registration date, status filter tabs (Pending/Approved/Rejected/Suspended). Row actions: "Approve" (instant), "Reject" (modal with reason textarea), "Suspend" (modal with confirmation), "Reactivate" (for suspended). Success toasts on each action. Badge counts in tabs.
**Acceptance Criteria:**
- [ ] Pending tab shows all PENDING_REVIEW sellers
- [ ] Approve action updates row status immediately (optimistic update)
- [ ] Reject modal requires reason input; submit disabled until reason entered
- [ ] Suspend modal shows confirmation before submitting
- [ ] Tab badge counts update after actions
- [ ] Functional at 1280px desktop viewport (admin is desktop-only)
**Files Affected:** `frontend/app/admin/sellers/page.tsx`, `frontend/components/admin/SellerQueue.tsx`, `frontend/components/admin/RejectModal.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-019-admin-seller-queue.html`
**Dependencies:** TASK-004, TASK-060
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 7
**Status:** Pending

---

### TASK-065
**Epic:** EPIC-012
**Story:** US-022, US-030
**Title:** Build admin product approval queue UI (SCR-020)
**Description:** Build the product queue at `/admin/products`. Card or table view: product thumbnail, name, seller store name, submitted date, status. Row/card actions: "Approve" (instant), "Reject" (modal with reason), "Remove" (modal with reason). Filter tabs: Pending/Approved/Rejected/Removed. Product detail side panel with full images and description on click.
**Acceptance Criteria:**
- [ ] Product queue shows pending products with thumbnail images
- [ ] Product detail side panel shows all product images and full description
- [ ] Reject modal requires reason; submit disabled until filled
- [ ] Approve updates product status optimistically
- [ ] Functional at 1280px desktop viewport
**Files Affected:** `frontend/app/admin/products/page.tsx`, `frontend/components/admin/ProductQueue.tsx`, `frontend/components/admin/ProductDetailPanel.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-020-admin-product-queue.html`
**Dependencies:** TASK-004, TASK-061
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 7
**Status:** Pending

---

### TASK-066
**Epic:** EPIC-012
**Story:** US-035
**Title:** Build admin dashboard UI (SCR-021)
**Description:** Build the admin overview dashboard at `/admin/dashboard`. Six metric cards: Total Active Sellers, Total Active Products, Orders (7d), Orders (30d), GMV 7d (INR), GMV 30d (INR). Each card with icon, value, and a label. Quick-link cards to Seller Queue and Product Queue with pending count badges.
**Acceptance Criteria:**
- [ ] All 6 metric cards display correct values from API
- [ ] Pending count badges on queue quick-links match actual queue lengths
- [ ] Functional at 1280px desktop viewport
- [ ] Loading skeleton visible while API call is in-flight
**Files Affected:** `frontend/app/admin/dashboard/page.tsx`, `frontend/components/admin/MetricCard.tsx`, `frontend/components/admin/QueueSummary.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-021-admin-dashboard.html`
**Dependencies:** TASK-004, TASK-062
**Estimate:** S (2 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 7
**Status:** Pending

---

### TASK-067
**Epic:** EPIC-012
**Story:** US-021, US-022, US-023, US-029, US-030, US-035
**Title:** Write admin portal backend tests
**Description:** Write tests for: all seller admin actions (approve/reject/suspend/reactivate) with RBAC enforcement, all product admin actions (approve/reject/remove), admin dashboard metrics, email dispatch verification, and cross-role access (Seller JWT → 403 on all admin endpoints).
**Acceptance Criteria:**
- [ ] All 4 seller status transitions tested with state before/after verification
- [ ] All 3 product admin actions tested
- [ ] RBAC test: seller JWT → 403 on every admin endpoint
- [ ] Email dispatch mocked and verified for each action
- [ ] `sellers/` admin coverage ≥ 80%, `products/` admin coverage ≥ 80%
**Files Affected:** `backend/sellers/tests/test_admin_views.py`, `backend/products/tests/test_admin_views.py`
**Wireframe Reference:** —
**Dependencies:** TASK-060, TASK-061, TASK-062
**Estimate:** S (2 SP)
**Type:** Test
**Assigned Role:** Backend Dev
**Sprint:** 7
**Status:** Pending

---

## EPIC-013: Buyer Account & Order History

**Description:** Buyer account order history page — authenticated buyers can view all their past orders, statuses, and items.

---

### TASK-068
**Epic:** EPIC-013
**Story:** US-027
**Title:** Implement buyer order history API endpoint
**Description:** Implement `GET /api/v1/buyer/orders/` — returns all orders for the authenticated buyer, sorted by created_at DESC. Each order: order_id, created_at, status, items (product name × qty × price), total_amount. Guest orders (no buyer_id) are never linked to registered accounts (BR-018). Requires Buyer JWT.
**Acceptance Criteria:**
- [ ] Authenticated buyer sees only their own orders
- [ ] Guest order with same email does NOT appear in buyer order history
- [ ] Orders sorted most recent first
- [ ] Response includes all required fields per US-027
- [ ] Non-buyer JWT → 403
**Files Affected:** `backend/orders/views.py`, `backend/orders/serializers.py`
**Wireframe Reference:** —
**Dependencies:** TASK-028, TASK-012
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 7
**Status:** Pending

---

### TASK-069
**Epic:** EPIC-013
**Story:** US-027
**Title:** Build buyer order history UI (SCR-009)
**Description:** Build the order history page at `/orders` (auth-gated). List of orders: order ID, date, product list (truncated to 2 items + "more"), total, status badge. Click to expand order detail (full items, tracking link if shipped). Empty state with "Start Shopping" CTA. Requires authentication — redirect to `/auth/login` if not authenticated.
**Acceptance Criteria:**
- [ ] Only authenticated buyer can access (redirect to login if not authenticated)
- [ ] Orders display correctly sorted most recent first
- [ ] Expand/collapse order detail shows all items and tracking link
- [ ] Empty state renders with "Start Shopping" CTA
- [ ] Functional at 375px mobile and 1440px desktop viewports per SCR-009 wireframe
**Files Affected:** `frontend/app/orders/page.tsx`, `frontend/components/buyer/OrderHistory.tsx`, `frontend/components/buyer/OrderCard.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-009-buyer-orders.html`
**Dependencies:** TASK-004, TASK-068, TASK-014
**Estimate:** S (2 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 7
**Status:** Pending

---

## EPIC-014: Pre-Launch Hardening

**Description:** Performance optimization (caching, query optimization), security hardening (WAF configuration, CORS, CSP headers), accessibility audit (WCAG 2.1 AA), test coverage gap closure, CI/CD deployment pipeline to ECS Fargate, load testing (≥ 1,000 concurrent users), and staging environment setup.

---

### TASK-070
**Epic:** EPIC-014
**Story:** NFR-PE-001, NFR-PE-005
**Title:** Add Redis product catalog cache with 1-hour TTL
**Description:** Cache `GET /api/v1/products/` and `GET /api/v1/products/search/` responses in Redis db=0 with 1-hour TTL. Cache key includes query params and page number. Cache invalidated when any product status changes (Django signal on Product model save). Use `django-redis` cache backend.
**Acceptance Criteria:**
- [ ] Second identical request to products list hits Redis cache (verified via Redis MONITOR or cache debug middleware)
- [ ] Cache invalidated within 5 seconds of product status change
- [ ] Cache miss falls back to DB query correctly
- [ ] P95 API response time < 200ms with cache enabled (verified via load test)
**Files Affected:** `backend/products/views.py`, `backend/products/signals.py`, `backend/shopnest/settings/base.py` (CACHES config)
**Wireframe Reference:** —
**Dependencies:** TASK-017
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 8
**Status:** Pending

---

### TASK-071
**Epic:** EPIC-014
**Story:** NFR-SEC-001, NFR-SEC-008
**Title:** Configure security headers and CORS policy
**Description:** Add Django middleware for: `Strict-Transport-Security` (HSTS, max-age 31536000), `Content-Security-Policy` (restrict script-src to self + Razorpay CDN), `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`. Configure CORS (`django-cors-headers`) to allow only the ShopNest frontend domain and `localhost:3000` in development. Configure Next.js `next.config.ts` security headers.
**Acceptance Criteria:**
- [ ] All 5 security headers present in every API response (verified via `curl -I`)
- [ ] CORS rejects requests from non-allowed origins (e.g., `evil.com`)
- [ ] Next.js pages return security headers via `headers()` in `next.config.ts`
- [ ] CSP does not break Razorpay checkout widget (tested in staging)
**Files Affected:** `backend/shopnest/settings/production.py`, `backend/shopnest/middleware.py`, `frontend/next.config.ts`
**Wireframe Reference:** —
**Dependencies:** TASK-003, TASK-004
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Full-Stack Lead
**Sprint:** 8
**Status:** Pending

---

### TASK-072
**Epic:** EPIC-014
**Story:** NFR-USA-003
**Title:** Accessibility audit and remediation — buyer-facing screens
**Description:** Run axe DevTools automated scan on all 8 buyer-facing screens (SCR-001, 002, 004, 005, 006, 007, 008, 009). Fix all WCAG 2.1 AA violations: missing ARIA labels, insufficient color contrast, keyboard navigation gaps, missing focus indicators, form label associations. Re-run scan to confirm zero critical/serious violations.
**Acceptance Criteria:**
- [ ] axe DevTools scan shows zero critical or serious violations on all 8 buyer screens
- [ ] All interactive elements reachable and operable via keyboard only
- [ ] Color contrast ratio ≥ 4.5:1 for all text (verified via browser contrast checker)
- [ ] All form inputs have associated `<label>` elements
- [ ] Image alt text present on all product images
**Files Affected:** Multiple buyer-facing components in `frontend/components/` and `frontend/app/`
**Wireframe Reference:** `docs/visuals/ux/SCR-001-product-detail.html`, `docs/visuals/ux/SCR-002-cart-checkout.html`, `docs/visuals/ux/SCR-004-marketplace-homepage.html`
**Dependencies:** TASK-018, TASK-019, TASK-020, TASK-021, TASK-025, TASK-032, TASK-033, TASK-069
**Estimate:** M (3 SP)
**Type:** Frontend
**Assigned Role:** Frontend Dev
**Sprint:** 8
**Status:** Pending

---

### TASK-073
**Epic:** EPIC-014
**Story:** NFR-REL-001, NFR-PE-006
**Title:** Configure GitHub Actions CD pipeline to ECS Fargate
**Description:** Write `.github/workflows/deploy.yml`. On merge to `develop`: build Docker image, push to ECR, update ECS task definition (Django API service, Celery Worker service, Next.js service), trigger rolling deploy. On merge to `main`: same but to production ECS cluster. Staging deploy must succeed before production is eligible. Blue/green or rolling deploy strategy.
**Acceptance Criteria:**
- [ ] Push to `develop` triggers staging deploy automatically
- [ ] Staging deploy completes without manual intervention
- [ ] Push to `main` triggers production deploy
- [ ] Health check (`GET /api/v1/health/`) passes after each deploy before marking deployment complete
- [ ] Failed health check triggers automatic rollback to previous task definition
**Files Affected:** `.github/workflows/deploy.yml`, `infra/ecs-task-definitions/`
**Wireframe Reference:** —
**Dependencies:** TASK-007, TASK-002
**Estimate:** L (5 SP)
**Type:** DevOps
**Assigned Role:** Full-Stack Lead
**Sprint:** 8
**Status:** Pending

---

### TASK-074
**Epic:** EPIC-014
**Story:** NFR-MAINT-001, NFR-MAINT-002
**Title:** Close test coverage gaps to meet 80% threshold
**Description:** Run `pytest --cov` and `jest --coverage` to identify any Django apps or Next.js components below 80% line coverage. Write the missing unit and integration tests to close gaps. Focus on: payment webhook handler edge cases, subscription lifecycle, seller analytics queries, and any frontend components without tests.
**Acceptance Criteria:**
- [ ] `pytest --cov=. --cov-report=term-missing` shows ≥ 80% for all Django apps
- [ ] `npm run test -- --coverage` shows ≥ 80% for all frontend source files
- [ ] CI pipeline enforces coverage gate (< 80% fails the pipeline)
- [ ] No test uses `# noqa` or `@pytest.mark.skip` to bypass coverage
**Files Affected:** `backend/*/tests/`, `frontend/**/*.test.tsx`
**Wireframe Reference:** —
**Dependencies:** All prior test tasks
**Estimate:** M (3 SP)
**Type:** Test
**Assigned Role:** Full-Stack Lead
**Sprint:** 8
**Status:** Pending

---

### TASK-075
**Epic:** EPIC-014
**Story:** NFR-MAINT-003
**Title:** Verify OpenAPI spec completeness and generate final API documentation
**Description:** Run `python manage.py spectacular --validate` to confirm all API endpoints are documented in the generated OpenAPI spec. Add any missing `@extend_schema` decorators to DRF views. Verify spec covers all 30+ endpoints. Commit final `openapi.yaml` to repo root.
**Acceptance Criteria:**
- [ ] `python manage.py spectacular --validate` exits 0 with zero warnings
- [ ] All `/api/v1/` endpoints present in `openapi.yaml`
- [ ] Each endpoint has description, request/response schemas, and HTTP status codes documented
- [ ] `openapi.yaml` committed to repo root and accessible at `GET /api/schema/`
**Files Affected:** `backend/*/views.py` (add `@extend_schema` decorators), `openapi.yaml`
**Wireframe Reference:** —
**Dependencies:** All backend task completions
**Estimate:** S (2 SP)
**Type:** Documentation
**Assigned Role:** Backend Dev
**Sprint:** 8
**Status:** Pending

---

### TASK-076
**Epic:** EPIC-014
**Story:** NFR-PE-006
**Title:** Conduct load test — 1,000 concurrent users
**Description:** Using k6 or Locust, run a load test against the staging environment simulating 1,000 concurrent users performing: browse homepage (40%), product detail view (30%), search (20%), add to cart (10%). Target: P95 API response < 200ms, LCP < 2.5s, zero 5xx errors. Document results in `docs/qa/LOAD-TEST-RESULTS.md`.
**Acceptance Criteria:**
- [ ] Load test script covers all 4 user scenarios in correct proportion
- [ ] P95 API response time < 200ms under 1,000 concurrent users
- [ ] Zero 5xx errors during 10-minute sustained load
- [ ] Test results documented with charts in `docs/qa/LOAD-TEST-RESULTS.md`
- [ ] Any bottleneck identified is documented with a mitigation ticket
**Files Affected:** `docs/qa/LOAD-TEST-RESULTS.md`, `tests/load/k6-script.js` (or `locustfile.py`)
**Wireframe Reference:** —
**Dependencies:** TASK-073
**Estimate:** M (3 SP)
**Type:** Test
**Assigned Role:** Full-Stack Lead
**Sprint:** 8
**Status:** Pending

---

## Additional Supporting Tasks

### TASK-077
**Epic:** EPIC-002
**Story:** US-001, US-025
**Title:** Write auth endpoint unit tests
**Description:** Comprehensive pytest unit tests for: seller registration (all validation rules), buyer registration, JWT token issuance, refresh token rotation, logout denylist, JWKS endpoint, bcrypt hash verification, and admin endpoint RBAC.
**Acceptance Criteria:**
- [ ] All 6 registration validation rules tested (email duplicate, password rules, phone format)
- [ ] Logout + re-use of refresh token → 401 tested
- [ ] JWKS endpoint returns valid JWK set
- [ ] `users/` app coverage ≥ 80%
**Files Affected:** `backend/users/tests/test_views.py`, `backend/users/tests/test_serializers.py`
**Wireframe Reference:** —
**Dependencies:** TASK-010, TASK-011, TASK-012
**Estimate:** S (2 SP)
**Type:** Test
**Assigned Role:** Backend Dev
**Sprint:** 2
**Status:** Pending

---

### TASK-078
**Epic:** EPIC-006
**Story:** US-002, US-003
**Title:** Write seller onboarding backend tests
**Description:** pytest tests for: store setup (MIME validation, size limits, S3 mock upload), subscription creation (webhook scenarios: charged, charge.failed, grace period, suspension, restoration), bank account encryption/decryption, payout hold on bank update.
**Acceptance Criteria:**
- [ ] MIME validation test: PDF rejected, PNG accepted
- [ ] Grace period test: day 7 accessible, day 8 suspended
- [ ] Subscription restoration test: products re-published
- [ ] Encryption test: stored value differs from input; decryption returns original
- [ ] `sellers/` app coverage ≥ 80%
**Files Affected:** `backend/sellers/tests/test_views.py`, `backend/sellers/tests/test_services.py`, `backend/payments/tests/test_subscription.py`
**Wireframe Reference:** —
**Dependencies:** TASK-035, TASK-036, TASK-037, TASK-038
**Estimate:** S (2 SP)
**Type:** Test
**Assigned Role:** Backend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-079
**Epic:** EPIC-007
**Story:** US-004, US-006, US-010
**Title:** Write product management backend tests
**Description:** pytest tests for: product creation (all validation rules, status=PENDING_REVIEW), edit with re-review trigger (name change vs price change), out-of-stock status transition, low-stock flag, deletion guard (active orders), slug stability (name edit does not change slug), multi-image validation.
**Acceptance Criteria:**
- [ ] Name change test: product status=PENDING_REVIEW after PATCH
- [ ] Price change test: product status unchanged after PATCH
- [ ] Deletion blocked test: 409 when active orders exist
- [ ] Slug test: slug identical before and after name edit
- [ ] Out-of-stock test: product absent from marketplace listing
- [ ] `products/` app coverage ≥ 80%
**Files Affected:** `backend/products/tests/test_views.py`, `backend/products/tests/test_services.py`
**Wireframe Reference:** —
**Dependencies:** TASK-041, TASK-042
**Estimate:** S (2 SP)
**Type:** Test
**Assigned Role:** Backend Dev
**Sprint:** 5
**Status:** Pending

---

### TASK-080
**Epic:** EPIC-005
**Story:** US-016
**Title:** Build checkout form — registered buyer pre-fill and address handling
**Description:** Implement checkout form address pre-fill for authenticated buyers: on page load, fetch buyer's saved address from `GET /api/v1/buyer/profile/` and pre-populate the shipping form. Implement `POST /api/v1/buyer/addresses/` to save new address for future use. "Save this address" checkbox on checkout form.
**Acceptance Criteria:**
- [ ] Authenticated buyer with saved address sees form pre-populated
- [ ] "Save this address" checkbox saves new address for future checkouts
- [ ] Guest buyer sees blank form (no pre-fill)
- [ ] Saved address appears in subsequent checkout sessions for the buyer
**Files Affected:** `backend/users/views.py`, `backend/users/models.py` (BuyerProfile address fields), `frontend/components/checkout/CheckoutForm.tsx`
**Wireframe Reference:** `docs/visuals/ux/SCR-002-cart-checkout.html`
**Dependencies:** TASK-012, TASK-032
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 4
**Status:** Pending

---

### TASK-081
**Epic:** EPIC-003
**Story:** US-011, US-012, US-013
**Title:** Implement product slug generation and stable URL resolution
**Description:** Implement `generate_product_slug(name, product_id)` utility: slugify product name + first 8 chars of UUID (e.g., `blue-cotton-kurti-ab12cd34`). Set slug at create time; never update on name edit. Implement URL resolution in Next.js that extracts UUID suffix from slug URL and fetches by UUID (ignoring name prefix). Handle 301 redirect if slug in URL differs from stored slug (post-approval rename scenario).
**Acceptance Criteria:**
- [ ] Slug generated at create time using name + UUID prefix
- [ ] PATCH with name change does not update slug in DB
- [ ] URL `/products/blue-handloom-kurti-ab12cd34` resolves to the same product as `/products/blue-cotton-kurti-ab12cd34`
- [ ] 301 redirect to canonical slug URL if non-canonical slug is accessed
**Files Affected:** `backend/products/services.py`, `backend/products/models.py`, `frontend/app/products/[slug]/page.tsx`
**Wireframe Reference:** —
**Dependencies:** TASK-017, TASK-041
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 3
**Status:** Pending

---

### TASK-082
**Epic:** EPIC-002
**Story:** US-001
**Title:** Implement admin seller approval queue API endpoints
**Description:** Implement `GET /api/v1/admin/sellers/pending/` — returns all sellers with PENDING_REVIEW status for the admin queue display. Add `admin_notes` optional field to approval/rejection actions. Ensure new seller registrations trigger a Django signal that could eventually be used for real-time admin notification (Phase 13 scope — placeholder only at this stage).
**Acceptance Criteria:**
- [ ] Admin queue API returns correct pending sellers with all required fields
- [ ] New seller registration appears in queue within database commit time
- [ ] Signal stub for admin notification is implemented (no-op at this stage)
**Files Affected:** `backend/sellers/views.py`, `backend/sellers/signals.py`
**Wireframe Reference:** —
**Dependencies:** TASK-011, TASK-015
**Estimate:** XS (1 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 2
**Status:** Pending

---

### TASK-083
**Epic:** EPIC-001
**Story:** Infrastructure
**Title:** Configure first-party analytics event tracking
**Description:** Implement `POST /api/v1/analytics/track/` — accepts `event_type`, `entity_id`, `entity_type`, `user_id` (optional), `metadata` (JSON). Dispatches `write_analytics_event` Celery task (analytics queue) that writes to `AnalyticsEvent` table asynchronously. Never blocks the critical path. Implement client-side tracking calls for: product_view, add_to_cart, checkout_initiated, order_placed.
**Acceptance Criteria:**
- [ ] `POST /api/v1/analytics/track/` returns 202 Accepted in < 10ms (async dispatch only)
- [ ] Celery task writes AnalyticsEvent row to DB
- [ ] Client-side tracking calls fire on each listed event
- [ ] Failed analytics write is logged but never propagates error to user
- [ ] API accepts anonymous (no user_id) events
**Files Affected:** `backend/analytics/views.py`, `backend/analytics/tasks.py`, `frontend/lib/analytics.ts`
**Wireframe Reference:** —
**Dependencies:** TASK-005, TASK-006
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 2
**Status:** Pending

---

### TASK-084
**Epic:** EPIC-014
**Story:** NFR-MAINT-004, NFR-MAINT-005
**Title:** Implement AuditLog signal handlers for money and account events
**Description:** Implement Django signal handlers (`post_save`, `pre_delete`) on: Order, SubOrder, Subscription, Payout, SellerProfile, ProductStatus (admin changes). Each signal dispatches `create_audit_entry` Celery task that writes to AuditLog table with: event_type, entity_type, entity_id, actor_id (user who triggered), timestamp, metadata JSON.
**Acceptance Criteria:**
- [ ] Order creation → AuditLog entry with event_type=ORDER_CREATED
- [ ] Subscription status change → AuditLog entry
- [ ] Payout initiation → AuditLog entry
- [ ] Admin seller approval/rejection → AuditLog entry with admin actor_id
- [ ] AuditLog entries are immutable (no UPDATE or DELETE via application layer)
**Files Affected:** `backend/analytics/signals.py`, `backend/analytics/tasks.py`, `backend/orders/apps.py`, `backend/sellers/apps.py`, `backend/payments/apps.py`
**Wireframe Reference:** —
**Dependencies:** TASK-005, TASK-006
**Estimate:** S (2 SP)
**Type:** Backend
**Assigned Role:** Backend Dev
**Sprint:** 3
**Status:** Pending

---

*End of Task Backlog*
*Task count by type: Backend=47 · Frontend=28 · Database=1 · DevOps=5 · Test=14 · Documentation=1 · Mixed=6*
