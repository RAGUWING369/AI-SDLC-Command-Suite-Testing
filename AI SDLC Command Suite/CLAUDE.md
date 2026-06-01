# CLAUDE.md — SOTA SDLC Suite · Project Context

> **Priority:** Highest. Overrides Claude's built-in defaults for this project.

> **Agent Behavior Rules:** All phase agents (01–14) are governed by two rule files in `rules/`:
> - `rules/RULE-BEHAVIOR.md` — Pre-execution rules: zero hallucination, context gap scanning, question format, context hierarchy, missing prerequisite protocol
> - `rules/RULE-EXECUTION.md` — Execution rules: strict execution order, assumption logging, mid-phase gap handling, human gate compliance, phase completion & artifact synchronization (Rule 11)
> Every agent reads **both files** as Step 0, in that order, before any other context file or action.
> **Do not modify either rule file without team review — changes affect all 14 phase agents.**
> **You Must:** With each SDLC phase completion or modifications made while SDLC phase completion, Update this CLAUDE.md file accordingly to make it prefectly aligned so that teams can come back review this file.

> **AI SDLC Project Type:** New greenfield project: `/sdlc:ideate`

---

## Suite Setup Status

| Tool | Status | Notes |
|------|--------|-------|
| Repowise | [x] Not Needed - greenfield, no existing codebase | Skip entirely for greenfield |
| Claude Code | [x] Active | Primary AI coding assistant |
| Context Bridge | [x] Not needed - single repository | Single monorepo architecture |

---

## Project Identity

**Project Name:** ShopNest

**Project Type:** 
```
[x] Greenfield — new project from scratch
```

**Repository Architecture:**
```
[x] Single Repo (Monorepo)
```

**Current Phase:** 7. Implementation

**Repository URL(s):**
- Primary: https://github.com/RAGUWING369/AI-SDLC-Command-Suite-Testing.git

**Started:** [2026-04-22]

---

## Project Description
> ShopNest is a lightweight, full-featured e-commerce web application modelled after platforms like Amazon and Flipkart, designed for small-to-mid-sized online retailers (50–500 products, 100–1,000 daily active users). It enables sellers to list and manage products and enables buyers to browse, search, add to cart, checkout, and track orders — all through a fast, mobile-responsive web interface backed by a robust Django REST API.

---

## Target Users

**Buyers:** General consumers aged 18–50 who shop online regularly. Low-to-moderate technical proficiency. They expect fast page loads, intuitive navigation, a reliable checkout experience, and clear order tracking.

**Sellers/Admins:** Small business owners and store managers who manage product listings, inventory, and order fulfilment through an admin dashboard. Moderate technical proficiency. They need a clean interface to add/edit products and view order status.

**User Scale:** 500–1,000 active users (mid-market)

---

## Technology Stack

### Target Stack (greenfield)

| Layer | Technology | Version | Notes |
|-------|-----------|---------|-------|
| Frontend Language | TypeScript | 5.x | Strict mode enabled |
| Frontend Framework | Next.js | 14 (App Router) | SSR + SSG for product pages; CSR for cart/checkout/dashboard |
| Backend Language | Python | 3.12 | PEP 8 + Black + isort |
| Backend Framework | Django + DRF | 5.x + 3.15 | REST API; 7 modular apps |
| WSGI Server | Gunicorn | 21.x | 4 workers × 4 threads per ECS task |
| UI Library | Tailwind CSS | 3.x | Utility-first styling |
| Database | PostgreSQL | 16 | Primary data storage; UUID PKs; GIN FTS index |
| State Management | Zustand | 4.x | Cart, Session state |
| Cache / Broker | Redis | 7.x | Cart (db=0), Celery broker (db=1), JWT denylist (db=2) |
| Async Tasks | Celery + Celery Beat | 5.x | Weekly payout, email dispatch, analytics writes |
| Cloud | AWS ap-south-1 | NA | Mumbai region — DPDPA data localization |
| Object Storage | AWS S3 | NA | Product images + GST invoice PDFs |
| CDN | AWS CloudFront | NA | Static assets + images; Mumbai/Delhi/Chennai edges |
| Compute | AWS ECS Fargate | NA | Serverless containers; 3 ECS services |
| Container Registry | AWS ECR | NA | Private registry with image scanning |
| Load Balancer | AWS ALB | NA | Path routing; TLS termination; WAF-attached |
| WAF | AWS WAF | NA | GeoIP India-only; SQLi/XSS; rate limiting |
| Payment Gateway | Razorpay | NA | Buyer checkout (UPI, cards, NetBanking, EMI) + seller subscriptions + weekly payouts |
| Email | AWS SES | NA | Transactional email (order, payout, subscription) |
| Monitoring | AWS CloudWatch + X-Ray | NA | Logs, metrics, alarms, distributed tracing |
| API Docs | drf-spectacular | 0.27.x | OpenAPI 3.0 spec generation |
| Auth | djangorestframework-simplejwt | 5.3.x | RS256 JWT; access 24h; refresh 30d |
| CI/CD | GitHub Actions | NA | Build → test → ECR push → ECS deploy |
| Containerization | Docker + Compose | 24.x + 2.x | Single image; different CMD per service |
| PDF Generation | WeasyPrint | 60.x | GST invoice PDFs |
| Field Encryption | PyCA cryptography | 41.x | AES-256-GCM for bank account numbers |

---

## Repository Structure
```
[To be scaffolded — /sdlc:implement will generate this]
Expected structure after scaffolding:

shopnest/
├── frontend/                  ← Next.js TypeScript app (App Router)
│   ├── app/                   ← Pages: SSR (product/category) + CSR (cart/checkout/dashboard)
│   ├── components/            ← UI components (shadcn/ui + Tailwind)
│   ├── lib/                   ← API client, Zustand stores, utilities
│   └── public/                ← Static assets
├── backend/                   ← Django REST API (Modular Monolith)
│   ├── shopnest/              ← Django project config (settings, urls, wsgi, celery)
│   ├── users/                 ← Auth, BuyerProfile, User model; RS256 JWT endpoints
│   ├── sellers/               ← SellerProfile, Store, StoreCategory, PayoutDetails
│   ├── products/              ← Product, ProductImage, Category; FTS search_vector
│   ├── orders/                ← Order, SubOrder, OrderLineItem, AuditLog
│   ├── cart/                  ← Cart (Redis-backed); guest session handling
│   ├── payments/              ← Razorpay checkout, subscriptions, payouts, webhook; SettlementLedgerEntry, Payout, WebhookIdempotencyLog, Invoice, Subscription
│   └── analytics/             ← analytics_events table + POST /api/v1/analytics/track
├── infra/                     ← ECS task definitions, VPC config, ALB rules 
├── docker-compose.yml         ← Local: postgres, redis, django, celery, next.js
├── Dockerfile                 ← Single image; CMD varies per ECS service
├── .github/
│   └── workflows/
│       ├── ci.yml             ← Test + lint + coverage gate
│       └── deploy.yml         ← ECR push + ECS rolling deploy
└── CLAUDE.md
```
---

## Key Commands
```
# ── Frontend (frontend/) ─────────────────────────────
# Install dependencies
npm install

# Start dev server (http://localhost:3000)
npm run dev

# Run unit tests (Jest + React Testing Library)
npm run test

# Run linter (ESLint Airbnb TypeScript config)
npm run lint

# Type check
npm run type-check

# Build for production
npm run build

# ── Backend (backend/) ──────────────────────────────
# Install dependencies
pip install -r requirements.txt

# Run dev server (http://localhost:8000)
python manage.py runserver

# Run migrations
python manage.py migrate

# Create new migration after model changes
python manage.py makemigrations

# Create superuser (admin access)
python manage.py createsuperuser

# Run tests with coverage
pytest --cov=. --cov-report=term-missing

# Start Celery worker (development)
celery -A shopnest worker -Q default,email,analytics,payouts -l INFO

# Start Celery Beat scheduler (development)
celery -A shopnest beat -l INFO --scheduler django_celery_beat.schedulers:DatabaseScheduler

# ── Docker (full stack local) ────────────────────────
# Start all services (postgres, redis, django, celery, next.js)
docker compose up

# Rebuild after dependency changes
docker compose up --build

# Run migrations inside container
docker compose exec backend python manage.py migrate

# ── Deploy (CI/CD via GitHub Actions) ───────────────
# Staging deploy triggered automatically on push to develop branch
# Production deploy triggered automatically on push to main branch
# Manual deploy: GitHub Actions → deploy.yml → Run workflow
```
---

## Coding Standards

- **Style Guide(Frontend):** Airbnb TypeScript ESLint config
- **Style Guide(Backend):** PEP 8 + Black formatter + isort
- **Commit Convention:** Conventional Commits (feat:, fix:, docs:, chore:, refactor:)
- **Branch Strategy:** `feature/xxx` → `develop` → `main`
- **PR Requirements:** 1 peer approval + CI pipeline green + no open review comments
- **Test Coverage Minimum:** Minimun 80% for new code (both frontend and backend)
- **Code Review:** Mandatory before any merge to develop or main
- **API Style:** RESTful, versioned (/api/v1/), snake_case JSON fields

---

## Architecture Decisions

**Architecture artifacts:** `docs/design/ARCHITECTURE.md` | `docs/design/DATA-MODEL.md` | `docs/design/API-SPEC.md` | `docs/design/TECH-STACK.md` | `docs/design/SECURITY-ARCHITECTURE.md`

**Architecture Decision Records:** `docs/design/adrs/`
- ADR-001: Modular Django Monolith Architecture Pattern
- ADR-002: Technology Stack Selection
- ADR-003: PostgreSQL 16 as Primary Database
- ADR-004: RS256 JWT Authentication Strategy
- ADR-005: AWS ECS Fargate Deployment Platform
- ADR-006: Celery + Redis for Async Task Processing
- ADR-007: PostgreSQL Full-Text Search for Product Discovery
- ADR-008: First-Party Analytics — No Third-Party SDK

Key decisions already made:
- **Multi-vendor marketplace confirmed** (2026-05-04): ShopNest is a multi-vendor platform — multiple independent sellers onboard, list products, and manage orders. Not a single-seller store.
- **Payment processor: Razorpay** (2026-05-04): Razorpay chosen for all buyer checkout (UPI, cards, NetBanking, EMI) and seller subscription billing. Stripe is NOT used.
- **Geography: India only** (2026-05-04): MVP targets India exclusively — INR pricing, Razorpay-native payments, Indian regulatory compliance.
- **Revenue model: SaaS subscription** (2026-05-04): Sellers pay a flat monthly subscription (price TBD, ~₹1,999/month — requires validation). No per-transaction commission.
- **Guest checkout in MVP** (2026-05-04): Buyers can complete purchase without creating an account.
- **Zero commission model — terminology clarified** (2026-05-05, PRD C-001 resolution): ShopNest charges zero *platform* commission. Razorpay payment gateway fee (~2%+GST) is absorbed by ShopNest as an operating cost, not passed to sellers. Sellers receive order amount net of gateway fee. Payout ledger must store both gross_amount_paise and gateway_fee_paise per sub-order.
- **Canonical product/seller status term** (2026-05-05, PRD C-002 resolution): Pending state for both seller accounts and product listings is uniformly called **"Pending Review"** across all artifacts and the codebase. Not "Pending Admin Approval."
- **First-party analytics only — no third-party SDK** (2026-05-05): All analytics events captured to internal `analytics_events` PostgreSQL table via `POST /api/v1/analytics/track`. No Mixpanel, Segment, or Amplitude in MVP. Analytics events written asynchronously via Celery to never block the critical path.
- **All monetary values stored as integer paise** (2026-05-05): ₹1 = 100 paise. All price, amount, and payout fields in DB and analytics use integer paise. Consistent with Razorpay API convention.
- **Feature flags as environment variables** (2026-05-05): Three MVP feature flags (`ff_payout_live`, `ff_product_admin_approval`, `ff_guest_checkout`) implemented as Django settings/env vars. No LaunchDarkly or dedicated flag service at MVP scale.
- **Rejected seller email block** (2026-05-05, PRD G-001 resolution): Rejected seller email addresses are blocked from re-registration via a `registration_blocked` status field on the seller profile. Sellers must contact support to appeal.
- **Modular Django Monolith** (2026-05-14, ADR-001): Single Django project; 7 apps with strong ORM boundaries; single ECS container image; CON-007 explicitly excludes microservices. Post-MVP extraction path: payments → analytics → in that order.
- **RS256 JWT authentication** (2026-05-14, ADR-004): Asymmetric RSA-2048 key pair; djangorestframework-simplejwt; access token 24h / refresh 30d; bcrypt cost 12; refresh token denylist in Redis; JWKS endpoint at `/.well-known/jwks.json`.
- **ECS Fargate ap-south-1** (2026-05-14, ADR-005): 3 ECS services (Django API, Celery Worker, Next.js); ALB path routing (single ALB); Multi-AZ ap-south-1a + 1b; WAF GeoIP India-only; estimated ~$220/month at MVP scale.
- **PostgreSQL FTS — no Elasticsearch at MVP** (2026-05-14, ADR-007): GIN-indexed tsvector column on products; P95 < 100ms at 50K products; eliminates ~$150-300/month OpenSearch cost; upgrade path: change one function (`ProductRepo.search()`) if needed post-MVP.
- **Triple-Gate Visibility Rule** (2026-05-14): Product appears in marketplace only when Seller.status=ACTIVE AND Subscription.status=ACTIVE AND Product.status=ACTIVE. All three gates enforced in product listing queryset.
- **19 database entities** (2026-05-14, DATA-MODEL.md): All with UUID PKs (gen_random_uuid()). Key entities: User, SellerProfile, Store, BuyerProfile, PayoutDetails, Category, Product, ProductImage, Subscription, Invoice, Order, SubOrder, OrderLineItem, SettlementLedgerEntry, Payout, AuditLog, AnalyticsEvent, WebhookIdempotencyLog, StoreCategory.
- **Field-level encryption on PayoutDetails.account_number** (2026-05-14, SECURITY-ARCHITECTURE.md): AES-256-GCM using PyCA cryptography library; key in AWS Secrets Manager; defense-in-depth above RDS KMS encryption.
- **WebhookIdempotencyLog** (2026-05-14, DATA-MODEL.md): Unique constraint on `razorpay_webhook_id` prevents duplicate payment processing (NFR-REL-005). 90-day retention then purged.

---

## Non-Functional Requirements

| Requirement | Target | Current |
|-------------|--------|---------|-
| API Response Time (P95) | < 200ms | TO BE MEASURED (Phase 9) |
| Page Load — LCP (mobile 4G) | < 2.5s | TO BE MEASURED (Phase 9) |
| First Contentful Paint | < 1.5s | TO BE MEASURED (Phase 9) |
| Uptime SLA | ≥ 99.9% monthly | TO BE MEASURED (Phase 13) |
| RTO | < 1 hour | TO BE MEASURED |
| RPO | < 1 hour | TO BE MEASURED |
| Product Image Load (CloudFront, ≤500KB) | < 1s | TO BE MEASURED (Phase 9) |
| Search Response Time (P95, ≤50K products) | < 500ms | TO BE MEASURED (Phase 9) |
| Concurrent Users (no degradation) | ≥ 1,000 | TO BE MEASURED (Phase 9) |
| Checkout Flow Completion | < 3 mins end-to-end | TO BE MEASURED (Phase 9) |
| DB Query Latency (P95) | < 50ms | TO BE MEASURED (Phase 9) |
| Data Retention | 7 years (minimum, all financial records) | Policy defined (REQUIREMENTS.md §5.1) |
| Test Coverage | ≥ 80% (backend + frontend) | TO BE MEASURED (Phase 9) |
| Payment Success Rate | ≥ 97% | TO BE MEASURED (Phase 9) |

---

## Constraints
- Budget: $2,000/month AWS cloud budget (MVP phase)
- Timeline: MVP by 2026-10-31 — core buyer + seller flows only
- Compliance: PCI-DSS handled by Razorpay — ShopNest never stores raw card data. GST compliance required pre-launch (subscription invoicing). Indian Consumer Protection (E-Commerce) Rules 2020 apply as a marketplace.
- Team Size: 3 developers (1 full-stack lead, 1 frontend, 1 backend)
- Parallel operation required: No — greenfield, no existing system
- Rollback window: Not applicable — no legacy system

---

## Phase Artifacts Index

| Phase | Status | Primary Artifact | Last Updated |
|-------|--------|-----------------|--------------|
| 1. Ideation | ✅ Complete | `docs/ideation/PROJECT-CONCEPT.md` | 2026-05-04 |
| 2. Requirements | ✅ Complete | `docs/requirements/REQUIREMENTS.md` | 2026-05-04 |
| 3. PRD | ✅ Complete | `docs/prd/PRD.md` | 2026-05-05 |
| 4. Architecture | ✅ Complete | `docs/design/ARCHITECTURE.md` | 2026-05-14 |
| 5. UX Design | ✅ Complete | `docs/ux/USER-JOURNEYS.md` | 2026-05-15 |
| 6. Task Breakdown | ✅ Complete | `docs/planning/TASKS.md` | 2026-05-19 |
| 7. Implementation | [TO BE UPDATED] | `src/` | |
| 8. Code Review | [TO BE UPDATED] | PRs in GitHub/GitLab | TO BE UPDATED |
| 9. Testing | [TO BE UPDATED] | `docs/qa/TEST-RESULTS.md` | |
| 10. Security | [TO BE UPDATED] | `docs/security/SECURITY-REVIEW.md` | TO BE UPDATED |
| 11. CI/CD | [TO BE UPDATED] | `.github/workflows/` | TO BE UPDATED |
| 12. Deployment | [TO BE UPDATED] | `docs/releases/` | TO BE UPDATED |
| 13. Monitoring | [TO BE UPDATED] | `docs/ops/MONITORING-SETUP.md` | TO BE UPDATED |
| 14. Retrospective | [TO BE UPDATED] | `docs/retros/` | TO BE UPDATED |

---

## Open Questions

| Question | Owner | Status |
|----------|-------|--------|
| Payment gateway — Stripe or Razorpay? | Team Lead | ✅ Closed — Razorpay confirmed (2026-05-04) |
| Multi-vendor marketplace or single seller? | Product Owner | ✅ Closed — Multi-vendor marketplace confirmed (2026-05-04) |
| Guest checkout required in MVP? | Product Owner | ✅ Closed — Yes, required in MVP (2026-05-04) |
| Target geography — India only or global? | Team Lead | ✅ Closed — India only confirmed (2026-05-04) |
| Platform revenue / monetisation model? | Product Owner | ✅ Closed — Monthly SaaS subscription from sellers confirmed (2026-05-04) |
| Subscription pricing tiers — single flat rate or tiered? | Product Owner | ✅ Closed — Single flat plan confirmed (2026-05-04) |
| GST invoicing scope — ShopNest subscription billing only, or seller-to-buyer invoices too? | Team Lead | ✅ Closed — Subscription billing only; sellers manage their own B2C invoicing (2026-05-04) |
| Logistics — carrier integration or seller-managed in MVP? | Team Lead | ✅ Closed — Sellers manage own shipping; AWB entered manually; carrier integration is post-MVP (2026-05-04) |
| Seller verification at onboarding — Aadhaar/GSTIN required? | Product Owner | ✅ Closed — Self-declaration for MVP; GSTIN optional (captured, not validated) (2026-05-04) |
| Order cancellation policy — who can cancel, what window? | Product Owner | ✅ Closed — Buyer can cancel before seller confirms only; no returns in MVP (2026-05-04) |
| Product listing approval — immediate or admin-approved? | Team Lead | ✅ Closed — Admin-approved before going live (2026-05-04) |
| Seller payout mechanism — Razorpay Route or settlement cycle? | Team Lead | ✅ Closed — Weekly settlement cycle via Razorpay Payouts API (2026-05-04) |
| Notification channels — email only or SMS/WhatsApp? | Product Owner | ✅ Closed — Email only via AWS SES for MVP (2026-05-04) |
| Subscription pricing validation — is ₹1,999/month acceptable to target sellers? | Product Owner | Open — Requires beta seller pricing interviews before launch (Q3 2026) |
| Razorpay Payouts API onboarding — has ShopNest's Razorpay account been enabled for Payouts feature? | Team Lead | Open — Must be resolved before Phase 7 (Implementation) |
| OQ-001 (PRD): Rejected seller rejection reason display — should rejection reason be shown in the seller dashboard or only communicated via email? | Product Owner | ✅ Closed — Both: shown inline in seller product list (SCR-003 State 1, expanded on Rejected rows) AND emailed via AWS SES. Design decision made in Phase 5 UX. (2026-05-14) |
| OQ-002 (PRD): Weekly payout execution day/time — confirm Monday 09:00 IST or specify alternative | Business Stakeholder | Open — Confirm before Phase 7 (Implementation) |
| OQ-003 (PRD): ₹1,999/month subscription price — confirm final price point before seller onboarding begins | Product Owner | Open — Beta seller pricing interviews required; Q3 2026 pre-launch |
| OQ-004 (PRD): Admin approval SLA — is there a target review turnaround time for seller and product approvals? | Product Owner | Open — Operational policy decision; affects admin dashboard design |
| OQ-005 (PRD): Consumer Protection (E-Commerce) Rules 2020 compliance — has a legal review been initiated? | Legal / Team Lead | Open — Must be resolved before GA launch |
| OQ-006 (PRD): Seller bank account details update flow — should existing payout be held or processed to old details when seller updates bank account mid-cycle? | Business Stakeholder | Open — Payout policy decision; Phase 7 (Implementation) |

---

## Human Gates Log

- The human is monitoring you in an IDE. They can see everything. They will catch your mistakes. Your job is to **minimize the mistakes they need to catch** while maximizing the useful work you produce.

- You have unlimited stamina. The human does not. Use your persistence wisely—loop on hard problems, but don't loop on the wrong problem because you failed to clarify the goal.

> Every phase requires explicit human approval (reply: `APPROVED`) before the next phase begins.
> Agents write approval here automatically after receiving APPROVED. Do not edit manually.

| Phase | Status | Approved By | Date | Conditions |
|-------|--------|-------------|------|------------|
| Phase 1 — Ideation | ✅ Approved | Stakeholder | 2026-05-04 | None |
| Phase 2 — Requirements Engineering | ✅ Approved | Stakeholder | 2026-05-04 | None |
| Phase 3 — PRD | ✅ Approved | Stakeholder | 2026-05-05 | None |
| Phase 4 — Architecture | ✅ Approved | Stakeholder | 2026-05-14 | None |
| Phase 5 — UX Design (Testing — 3 screens) | ✅ Approved | Stakeholder | 2026-05-14 | None |
| Phase 5 — UX Design (Full Production — 21 screens) | ✅ Approved | Stakeholder | 2026-05-15 | None |
| Phase 6 — Task Breakdown & Sprint Planning | ✅ Approved | Stakeholder | 2026-05-19 | None |

---

## Important Context for Agents
- This is a greenfield project — no legacy code exists anywhere
- Build a small-scale ecommerce platform similar to Amazon/Flipkart/Ajio in scope but scoped down for MVP: product browsing, cart, checkout, order tracking
- Backend: Python Django REST Framework — all business logic lives in Django
- Frontend: Next.js with TypeScript — SSR for product/category pages (SEO critical), client-side rendering for cart and checkout flows
- Database: PostgreSQL 16 — use UUID primary keys throughout, not integer IDs
- Cache: Redis for cart data (TTL 24h), product catalog cache (TTL 1h), sessions
- Images: All product images stored in AWS S3, served via CloudFront CDN — never store images in the database
- Payments: Razorpay only (UPI, cards, NetBanking, EMI) — NEVER store raw card data, delegate entirely to Razorpay. Also use Razorpay Subscriptions API for seller monthly billing.
- Architecture: Modular Django apps (products, orders, users, cart, payments, sellers) — do NOT suggest microservices, a well-structured Django monolith is correct for MVP scale. Multi-vendor data isolation enforced at ORM level using seller_id FK on all seller-scoped models.
- Team: 3 developers — keep task estimates realistic, no over-engineering
- MVP Scope ONLY: Do not suggest features beyond core buyer/seller flow.
Out of scope for MVP: reviews, recommendations, wishlists, affiliate, loyalty points, mobile app, logistics integration, multi-language — these are post-MVP.
- Multi-vendor is IN SCOPE for MVP: multiple sellers onboard independently, manage their own products and orders, and pay a monthly subscription.
- SEO is important — product pages and category pages must be server-side rendered for search engine indexing
- **Viewport requirement (2026-05-22):** ALL buyer-facing frontend UI screens must be fully functional and correctly laid out at BOTH 375px (mobile) AND 1440px (desktop) viewports, matching the wireframe specifications in `docs/visuals/ux/`. Seller dashboard screens are tablet-first (768px minimum per NFR-USA-002). Admin screens are desktop-only (1280px). Every frontend task acceptance criterion must include both 375px and 1440px checkboxes for buyer-facing pages.
- Mobile-first — majority of traffic is mobile; 375px is the primary breakpoint, but 1440px desktop layout is equally required for all buyer-facing pages

--- 

*This file is the source truth for the Project Completion. Update it accordingly when changes are being made (example: architectural decisions change, etc.).*
*It is a living document — accuracy matters more than completeness.*