# Technology Stack — ShopNest
**Phase:** 04 — Architecture & Design
**Generated:** 2026-05-14
**Status:** Draft — Awaiting Human Gate Approval

> Every technology choice traces to a project constraint, NFR, or PRD requirement. No technology was chosen without comparing at least two alternatives. See ADR-002 for the combined rationale narrative.

---

## Full Stack Table

| Layer | Technology | Version | Rationale | Alternatives Considered | Why Rejected |
|-------|-----------|---------|-----------|------------------------|-------------|
| **Frontend Language** | TypeScript | 5.x (strict mode) | Type safety catches integration bugs at compile time; required by CLAUDE.md; Airbnb ESLint config requires TS | JavaScript (plain) | No compile-time type checking; API contract errors surface at runtime instead of build |
| **Frontend Framework** | Next.js | 14 (App Router) | SSR for product/category pages (SEO + NFR-PE-002 LCP < 2.5s); CSR for cart/checkout (interactivity); App Router enables streaming SSR; required by CLAUDE.md | Nuxt.js (Vue) | Team uses React; Vue context switch costs ~4 weeks; no team proficiency |
| | | | | Remix (React) | Newer, smaller ecosystem; fewer Next.js integrations for Razorpay; less community support for India-specific use cases |
| | | | | CRA / Vite SPA | No SSR — product pages would fail SEO requirements; LCP target unachievable without SSR on mobile 4G |
| **UI Library** | Tailwind CSS | 3.x | Utility-first; produces minimal CSS bundle (~15KB purged); mobile-first responsive design with `sm:`/`md:`/`lg:` prefixes; NFR-USA-001/002 met out of the box | Material UI (MUI) | 40KB+ minified component library; over-designed for a marketplace; harder to customize to brand |
| | | | | Bootstrap 5 | Class-collision patterns; not utility-first; difficult to achieve pixel-perfect mobile layouts |
| | | | | Chakra UI | Good DX but requires component wrapping; larger bundle than Tailwind |
| **State Management** | Zustand | 4.x | Minimal boilerplate; cart state and auth state fit Zustand's simple slice pattern; no Redux ceremony; required by CLAUDE.md | Redux Toolkit | Overkill for 2 state slices (cart + auth); 3× more code than Zustand for same functionality |
| | | | | React Context + useReducer | Causes full re-renders of component tree on state change; cart updates visible to all components would trigger unnecessary renders |
| | | | | Jotai / Recoil | Atom-based; good for complex state but overkill for cart and session |
| **Backend Language** | Python | 3.12 | Required by CLAUDE.md; team proficiency; Django ecosystem; excellent AWS SDK support (boto3); PEP 8 + Black formatter enforced | Go | Faster execution but no Django-equivalent framework; team has no Go proficiency; 3-month ramp-up cost |
| | | | | Node.js (TypeScript) | Would allow full-stack TypeScript but splits team across two Django-unfamiliar backend setups; loses Django admin, ORM, migrations |
| | | | | Ruby on Rails | No team proficiency; smaller ecosystem for Indian fintech integrations |
| **Backend Framework** | Django | 5.x | Required by CLAUDE.md; ORM with migrations covers all 19 entities; Django admin for bootstrapping; signals for audit logging; proven at 100K-1M users scale | FastAPI | No built-in ORM, migrations, or admin panel; requires 3× more boilerplate for CRUD operations |
| | | | | Flask | Micro-framework requires assembling ORM + migration + auth + serialization manually; weeks of setup vs. Django batteries-included |
| | | | | Django Ninja | FastAPI-like DRF replacement; newer, smaller community; drf-spectacular (chosen) works with DRF, not Ninja |
| **REST API** | Django REST Framework | 3.15 | Required by CLAUDE.md; ViewSets + serializers cover all 58 endpoints cleanly; DRF permissions classes enable RBAC; drf-spectacular generates OpenAPI 3.0 spec (NFR-MAINT-003) | Tastypie | Older, less actively maintained; worse serializer validation than DRF |
| | | | | django-ninja | FastAPI-like, newer; lacks DRF's ecosystem maturity; drf-spectacular does not support it |
| **WSGI Server** | Gunicorn | 21.x | Production-proven Python WSGI server; 4 workers × 4 threads = 16 concurrent slots per container; NFR-PORT-003 (WSGI compatibility) | uWSGI | More complex configuration; harder to containerize correctly; Gunicorn is simpler and equally capable at MVP scale |
| | | | | Daphne (ASGI) | ShopNest does not use WebSockets or async Django views; ASGI overhead without benefit |
| **API Documentation** | drf-spectacular | 0.27.x | Auto-generates OpenAPI 3.0 spec from DRF views; NFR-MAINT-003 (100% endpoint coverage); Swagger UI built-in | drf-yasg | Generates OpenAPI 2.0 (Swagger 2.0) — older standard; drf-spectacular generates OpenAPI 3.0 |
| | | | | Manual openapi.yaml | Risk of drift from implementation; drf-spectacular keeps spec and code in sync |
| **Database** | PostgreSQL | 16 | Required by CLAUDE.md; UUID PKs native; JSONB for delivery_address and state snapshots; FTS with GIN index (ADR-007, NFR-PE-005); Row-level locking for atomic stock decrement (BR-008); AES-256 encryption via RDS; PITR backup (NFR-REL-003) | MySQL 8.0 | Full-text search is inferior to PostgreSQL FTS; JSONB not natively available; UUID PK support less mature |
| | | | | MongoDB | Document model poor fit for relational order→sub-order→line-item schema; ACID transactions across collections complex; team proficiency in relational DB |
| | | | | SQLite | Not suitable for multi-process ECS environment; not production-grade for concurrent writes |
| **Cache / Broker** | Redis | 7.x | Required by CLAUDE.md; cart storage (TTL 24h, NFR data); product catalog cache (TTL 1h); Celery message broker; session token denylist (logout) | Memcached | No data structures beyond simple key-value; cannot serve as Celery broker; no TTL-based cart merging |
| | | | | DynamoDB (ElastiCache DAX) | AWS-native but adds operational complexity; Redis has richer data types needed for cart |
| **Async Task Queue** | Celery | 5.x | De facto Python async task queue; weekly payout cron via Celery Beat; email dispatch; analytics writes; webhook processing; ADR-006 | Django Q | Smaller community; less production-tested at scale; fewer monitoring integrations |
| | | | | RQ (Redis Queue) | Simpler but no periodic tasks (Celery Beat equivalent); would require separate scheduler |
| | | | | AWS SQS + Lambda | Serverless — no persistent workers; Lambda cold start adds latency to time-sensitive webhook processing; increases AWS service dependencies |
| **Auth Library** | djangorestframework-simplejwt | 5.3.x | RS256 JWT support (ADR-004, NFR-SEC-003); token refresh; token denylist on logout; integrates cleanly with DRF | python-jose | Lower-level; requires more integration code with DRF |
| | | | | PyJWT | Same — lower-level library; simplejwt provides DRF integration out of the box |
| | | | | django-allauth | OAuth/social login focused; no RS256 JWT support without customization |
| **Cloud Provider** | AWS | — | Required by CLAUDE.md; ECS Fargate, RDS, ElastiCache, S3, CloudFront, SES, Secrets Manager, WAF all available in ap-south-1 (Mumbai) for India data residency (DPDPA) | GCP | No Razorpay-specific SDK advantage; team has AWS knowledge; India data residency available but less mature than AWS ap-south-1 |
| | | | | Azure | Same — less team familiarity; Azure DevOps integration advantage not relevant for GitHub-based project |
| **Compute** | AWS ECS Fargate | — | Serverless containers — no EC2 management (NFR-PORT-002); auto-scaling; pay-per-task; CON-001 budget compliant; ADR-005 | EC2 Auto Scaling | Requires OS patching, AMI management — violates NFR-PORT-002; higher operational burden for 3-person team |
| | | | | AWS Lambda | Cold starts unacceptable for P95 < 200ms API NFR; 15-minute max execution blocks payout Celery job |
| | | | | AWS EKS (Kubernetes) | Overkill for 3 services; cluster management overhead; $200+/month control plane cost eats into $2,000 budget |
| **Object Storage** | AWS S3 | — | Required by CLAUDE.md; 11 nines durability; native CloudFront integration; boto3 SDK; IAM task role auth (no key management) | GCS | Not AWS-native; cross-cloud data transfer costs; team AWS familiarity |
| | | | | Cloudinary | SaaS CDN with transformation; higher cost at scale; vendor lock-in for image storage |
| **CDN** | AWS CloudFront | — | Required by CLAUDE.md; Indian edge PoPs (Mumbai, Delhi, Chennai); native S3 origin; NFR-PE-004 (<1s image load) | Cloudflare | Excellent CDN but not AWS-native; requires additional DNS and certificate management |
| | | | | Fastly | Enterprise pricing; overkill for MVP image volumes |
| **Email** | AWS SES | — | Required by CLAUDE.md (A-02-003); India-native; cheapest transactional email ($0.10/1,000 emails); boto3 integration; DKIM/SPF support; bounce handling | SendGrid | $15–$50/month at MVP volume; not AWS-native; additional vendor |
| | | | | Mailgun | Similar to SendGrid; additional vendor; SES is AWS-native and fits CON-001 budget |
| **Payment Gateway** | Razorpay | — | Required by CLAUDE.md (CON-005); India's leading gateway; supports UPI, cards, NetBanking, EMI natively; Subscriptions API for seller billing; Payouts API for weekly settlement; Refunds API | Stripe | Not India-native; UPI support through Stripe is inferior to Razorpay; INR settlement delayed |
| | | | | PayU India | Fewer developer tools; documentation less comprehensive than Razorpay; webhook reliability historically lower |
| **Containerization** | Docker | 24.x + Compose 2.x | Industry standard OCI containers (NFR-PORT-001); single Dockerfile for Django + Celery (same image, different CMD); separate Dockerfile for Next.js; docker-compose for local dev | Podman | Compatible but less team familiarity; Docker Compose support less mature |
| **Container Registry** | AWS ECR | — | Native ECS integration; image scanning; IAM role auth; no separate registry credentials | Docker Hub | Public registry; requires credentials management; private repo costs; not AWS-native |
| **CI/CD** | GitHub Actions | — | Inferred from GitHub repository; marketplace actions for AWS deployment; free for public repos; 2,000 min/month free for private | CircleCI | Additional account management; GitHub Actions covers all CI needs |
| | | | | GitLab CI | Requires GitLab hosting; team uses GitHub |
| **Monitoring** | AWS CloudWatch + X-Ray | — | Native to AWS stack; CloudWatch Logs for structured JSON logs; Metrics + Alarms for scaling triggers (Phase 13); X-Ray for distributed tracing within AWS; no additional cost beyond compute | Datadog | $15+/host/month — adds ~$180/month to CON-001 budget for 3 services; overkill at MVP scale |
| | | | | New Relic | Similar cost concern; CloudWatch sufficient for MVP observability needs |
| **Testing — Backend** | pytest + pytest-cov + pytest-django | 7.x / 4.x | NFR-MAINT-001 (≥80% coverage); pytest-django provides Django test client and fixtures; pytest-cov generates coverage report for CI gate; `factory_boy` for test fixtures | Django TestCase only | Less feature-rich; no coverage integration; pytest is industry standard |
| **Testing — Frontend** | Jest + @testing-library/react + Istanbul | 29.x | NFR-MAINT-002 (≥80% coverage); Istanbul coverage report in CI; RTL encourages testing user behavior not implementation | Cypress | E2E only — not suitable for unit/integration coverage gates; supplement with E2E in Phase 9 |
| | | | | Playwright | Same as Cypress — E2E focus; Jest for unit coverage |
| **Linting — Frontend** | ESLint + Airbnb config | — | Required by CLAUDE.md; enforces TypeScript strict mode; catches common Next.js anti-patterns | StandardJS | No TypeScript support; weaker rules |
| **Linting — Backend** | Black + isort + flake8 | — | Required by CLAUDE.md (PEP 8 + Black formatter + isort); Black is opinionated (zero-config formatting); isort keeps imports organized; flake8 for lint | pylint | More verbose; slower; Black is sufficient with flake8 for additional rules |
| **Secrets Management** | AWS Secrets Manager | — | No secrets in code or environment files (Architectural Principle 5); IAM task role provides access without credential files; automatic rotation support | AWS Parameter Store | Less secure for sensitive values (plaintext option); Secrets Manager adds encryption, versioning, and rotation |
| | | | | HashiCorp Vault | Self-hosted — operational burden for 3-person team; AWS Secrets Manager is fully managed |
| **Image Processing** | python-magic | 0.4.x | Server-side MIME type validation (reads file magic bytes, not extension); NFR-SEC-009; prevents extension-spoofing attacks | Python Pillow | Can read image headers but less reliable for MIME spoofing detection; python-magic reads actual file magic bytes |
| **PDF Generation** | WeasyPrint | 60.x | GST invoice PDF generation (FR-SUBSCR-007); HTML+CSS to PDF; supports INR currency symbols; no external service dependency | ReportLab | Lower-level; requires more code for layout; WeasyPrint uses HTML/CSS templates (easier for developers) |
| | | | | Puppeteer/Playwright (headless) | Requires Node.js process from Python — cross-runtime complexity; WeasyPrint is pure Python |
| **Crypto — Field Encryption** | cryptography | 42.x (PyCA) | AES-256 encryption of PayoutDetails.account_number at application layer (in addition to RDS-level encryption); industry-standard library; FIPS-compliant | PyCryptodome | Less actively maintained; PyCA cryptography is preferred by Python security community |
| **Environment Config** | python-decouple | 3.8 | Reads settings from environment variables and .env files (local only — .env is gitignored); 12-Factor App compliant | django-environ | Equivalent functionality; python-decouple is lighter and more widely used |

---

## Runtime Package Summary

### Backend (`requirements.txt`)
```
django==5.x
djangorestframework==3.15.x
djangorestframework-simplejwt==5.3.x
drf-spectacular==0.27.x
celery==5.x
redis==5.x
boto3==1.x
python-magic==0.4.x
python-jose==3.x
cryptography==42.x
weasyprint==60.x
python-decouple==3.8.x
gunicorn==21.x
psycopg2-binary==2.9.x
factory-boy==3.x
pytest==7.x
pytest-django==4.x
pytest-cov==4.x
black==24.x
isort==5.x
flake8==7.x
python-json-logger==2.x
Pillow==10.x
```

### Frontend (`package.json` key dependencies)
```json
{
  "next": "14.x",
  "react": "18.x",
  "typescript": "5.x",
  "tailwindcss": "3.x",
  "zustand": "4.x",
  "axios": "1.x",
  "react-hook-form": "7.x",
  "zod": "3.x",
  "@testing-library/react": "14.x",
  "jest": "29.x",
  "eslint": "8.x",
  "eslint-config-airbnb": "19.x"
}
```
