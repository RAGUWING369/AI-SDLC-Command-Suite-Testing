# SECURITY-ARCHITECTURE.md — ShopNest Security Architecture

**Version:** 1.0
**Date:** 2026-05-14
**Phase:** 4 — System Architecture & Design
**Status:** Draft — Pending Human Gate Approval

---

## Table of Contents

1. [Security Architecture Overview](#1-security-architecture-overview)
2. [Threat Model](#2-threat-model)
3. [Authentication Architecture](#3-authentication-architecture)
4. [Authorization Architecture (RBAC)](#4-authorization-architecture-rbac)
5. [Data Classification & Encryption](#5-data-classification--encryption)
6. [Network Security](#6-network-security)
7. [Secrets Management](#7-secrets-management)
8. [Payment Security (PCI-DSS)](#8-payment-security-pci-dss)
9. [Regulatory Compliance](#9-regulatory-compliance)
10. [OWASP Top 10 (2021) Mitigation](#10-owasp-top-10-2021-mitigation)
11. [Security Testing Requirements](#11-security-testing-requirements)
12. [Incident Response](#12-incident-response)

---

## 1. Security Architecture Overview

ShopNest is a multi-vendor SaaS e-commerce marketplace operating exclusively in India. The security architecture is designed around five foundational principles:

1. **Zero card data storage** — PCI-DSS scope is fully delegated to Razorpay; ShopNest never handles raw card data
2. **Defence in depth** — every security control has at least one compensating control at a different layer
3. **Least privilege** — IAM roles, database users, and API permission classes grant minimum access required
4. **Data localization** — all user PII and behavioral data remains in AWS ap-south-1 (Mumbai) for DPDPA 2023 compliance
5. **Multi-tenant isolation** — all seller-scoped resources enforced at the Django ORM layer via `seller_id` foreign key filtering, not application-layer branching

### Security Control Inventory

| Control Domain | Primary Control | Compensating Control |
|---------------|-----------------|---------------------|
| Authentication | RS256 JWT (24h access / 30d refresh) | Refresh token denylist in Redis |
| Authorisation | DRF Permission Classes (role-based) | Django middleware role extraction from JWT claims |
| Data in transit | TLS 1.2+ enforced at ALB | HTTPS-only redirect at ALB listener |
| Data at rest | AWS KMS AES-256 on RDS + S3 | Field-level AES-256 on `PayoutDetails.account_number` |
| Payment data | No card data stored (Razorpay) | Razorpay HMAC-SHA256 webhook signature verification |
| Network isolation | 3-tier VPC (public/private/isolated) | Security groups deny all unspecified traffic |
| Secrets | AWS Secrets Manager | Container-level environment variable injection |
| Input validation | Django serializer validation | AWS WAF SQLi/XSS managed rules |
| Rate limiting | AWS WAF rate rules on auth endpoints | Django throttle classes (DRF) |
| Vulnerability scanning | ECR image scanning on push | Phase 10 DAST (OWASP ZAP) |

---

## 2. Threat Model

### 2.1 Assets and Trust Levels

| Asset | Sensitivity | Location | Trust Boundary |
|-------|-------------|----------|---------------|
| Buyer PII (name, email, phone, address) | High | RDS PostgreSQL (encrypted) | Private subnet only |
| Seller PII (name, GSTIN, store details) | High | RDS PostgreSQL (encrypted) | Private subnet only |
| Bank account numbers (`PayoutDetails.account_number`) | Critical | RDS PostgreSQL (field-level encrypted) | Private subnet + app layer |
| Payment data (card numbers, CVV) | N/A — never stored | Razorpay infrastructure | Razorpay's trust boundary |
| JWT private signing key (RS256) | Critical | AWS Secrets Manager | Container memory only |
| Razorpay webhook secret | Critical | AWS Secrets Manager | Container memory only |
| Product images | Low | S3 (public read via CloudFront) | Public |
| GST invoices | Medium | S3 (private, presigned URL) | Private — buyer/seller only |
| Analytics events | Low-Medium | RDS PostgreSQL | Private subnet |
| Audit logs | High | RDS PostgreSQL | Private subnet |

### 2.2 Threat Actors

| Actor | Capability | Motivation | Primary Attack Vectors |
|-------|-----------|------------|----------------------|
| External attacker | Moderate — script-based | Financial gain, data theft | SQL injection, credential stuffing, XSS, IDOR |
| Malicious buyer | Low-Moderate | Free products, fake orders | Race conditions (overselling), payment fraud, account takeover |
| Malicious seller | Low-Moderate | Fake listings, competitor disruption | Mass product spam, fake order manipulation |
| Compromised seller account | High (trusted credentials) | Data theft, financial fraud | IDOR to access other sellers' data |
| Insider (admin) | High | Data theft, platform abuse | Unauthorized data access via admin panel |

### 2.3 STRIDE Threat Analysis

| STRIDE Category | Top Threats | Mitigation |
|----------------|-------------|------------|
| **Spoofing** | JWT token theft; account impersonation | HTTPS-only; HttpOnly refresh token cookie; 24h access token TTL; bcrypt password storage |
| **Tampering** | Order amount manipulation; webhook replay | Razorpay HMAC-SHA256 webhook verification; WebhookIdempotencyLog; database-level CHECK constraints |
| **Repudiation** | Disputed transactions; admin action denial | AuditLog table (before/after state snapshots on all state changes); Razorpay payment records |
| **Information Disclosure** | IDOR — seller accessing another seller's data; API over-exposure | seller_id FK filtering in all ORM queries; DRF serializer field-level exclusion |
| **Denial of Service** | Auth endpoint flooding; checkout spam | AWS WAF rate limiting on /api/v1/auth/*; DRF throttle classes; WAF GeoIP India-only |
| **Elevation of Privilege** | JWT role claim tampering; admin endpoint access | RS256 signature verification; DRF `IsAdminUser` permission class; server-side role from JWT claim only |

---

## 3. Authentication Architecture

### 3.1 JWT Token Lifecycle

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AUTHENTICATION FLOW                               │
└─────────────────────────────────────────────────────────────────────┘

Registration / Login
─────────────────────────────────
Client ──POST /api/v1/auth/register──► Django (BCryptSHA256, cost=12)
       ◄── 201 + access_token (24h) + refresh_token (30d, HttpOnly cookie)

Authenticated Request
─────────────────────────────────
Client ──GET /api/v1/seller/products──► ALB → Django
        Authorization: Bearer <access_token>

Django Middleware:
  1. Extract JWT from Authorization header
  2. Verify RS256 signature using public key (in-memory)
  3. Check exp claim (reject if expired)
  4. Extract user_id, role, seller_status from claims
  5. Attach to request.user — no DB lookup required

Token Refresh
─────────────────────────────────
Client ──POST /api/v1/auth/token/refresh──► Django
        (refresh_token sent as HttpOnly cookie)

Django:
  1. Verify RS256 signature on refresh token
  2. Check refresh token JTI against Redis denylist
  3. If not in denylist → issue new access_token
  4. If in denylist → 401 Unauthorized

Logout
─────────────────────────────────
Client ──POST /api/v1/auth/logout──► Django
        Authorization: Bearer <access_token>

Django:
  1. Extract refresh token JTI from request body
  2. Add JTI to Redis denylist with TTL = remaining token lifetime
  3. Return 200 (clear HttpOnly cookie on client)
```

### 3.2 JWT Token Specifications

| Property | Access Token | Refresh Token |
|----------|-------------|---------------|
| Algorithm | RS256 | RS256 |
| Key | RSA-2048 private key (Secrets Manager) | Same key pair |
| Expiry | 24 hours | 30 days |
| Storage (client) | Memory (JavaScript variable) | HttpOnly cookie (Secure, SameSite=Strict) |
| Claims | `user_id`, `role`, `seller_status`, `email`, `exp`, `iat`, `jti` | `user_id`, `exp`, `iat`, `jti` |
| Denylist | Not applicable (stateless) | Redis set: `denylist:{jti}` with TTL |
| Rotation | On each refresh | Not rotated — same token until expiry or logout |

### 3.3 Password Security

| Control | Specification |
|---------|---------------|
| Algorithm | BCryptSHA256 (`django.contrib.auth.hashers.BCryptSHA256PasswordHasher`) |
| Cost factor | 12 (~250ms per hash on modern hardware) |
| Hash format | `$2b$12$<22-char-salt><31-char-hash>` |
| Pre-hashing | Django BCryptSHA256 pre-hashes with SHA256 to handle passwords > 72 bytes |
| Storage | `users_user.password` — only the hash, never plaintext |
| Transmission | Only over HTTPS — TLS terminated at ALB |

### 3.4 Guest Authentication

Guest buyers authenticate stateless order tracking via:
- `X-Session-ID` header (UUID assigned by backend on first cart interaction, returned as HttpOnly cookie)
- Guest order tracking URL: `GET /api/v1/orders/track/{tracking_token}` where `tracking_token` is HMAC-SHA256(order_uuid + `:` + guest_email, server_secret)
- Server secret stored in AWS Secrets Manager; validated without database lookup

### 3.5 JWKS Endpoint

`GET /.well-known/jwks.json` — Returns the RS256 public key in JWK format for future service-to-service token verification. Public endpoint, cached 24h by consumers.

---

## 4. Authorization Architecture (RBAC)

### 4.1 Role Definitions

| Role | Identifier in JWT | Assigned At | Description |
|------|------------------|-------------|-------------|
| Buyer | `role: "BUYER"` | Registration (buyer flow) | Browse, cart, checkout, order tracking |
| Seller | `role: "SELLER"` | Registration (seller flow) | Product management, order fulfillment, dashboard |
| Admin | `role: "ADMIN"` | Manual assignment by Platform Admin | Platform management, approvals, payout oversight |

### 4.2 DRF Permission Classes

| Class | Condition | Applied To |
|-------|-----------|------------|
| `IsAuthenticated` | Valid JWT present | All protected endpoints |
| `IsBuyerUser` | `role == 'BUYER'` | Buyer-specific endpoints (order history, profile) |
| `IsSellerUser` | `role == 'SELLER'` | Seller dashboard, product management |
| `IsAdminUser` | `role == 'ADMIN'` | Admin panel, approvals, platform analytics |
| `IsSellerWithActiveSubscription` | `role == 'SELLER'` AND `seller_status == 'ACTIVE'` | Seller product CRUD, order management |
| `IsOwnerOrAdmin` | Resource `seller_id == request.user.id` OR `role == 'ADMIN'` | Resource-level ownership check |
| `AllowAny` | No check | Public endpoints (product browse, search) |

### 4.3 RBAC Permissions Matrix

| Resource | Buyer | Seller (Active) | Seller (Pending/Suspended) | Admin |
|----------|-------|----------------|--------------------------|-------|
| `GET /api/v1/products/` | ✅ | ✅ | ✅ | ✅ |
| `GET /api/v1/products/{id}` | ✅ | ✅ | ✅ | ✅ |
| `POST /api/v1/seller/products/` | ❌ | ✅ | ❌ | ✅ |
| `PUT /api/v1/seller/products/{id}` | ❌ | ✅ (own products only) | ❌ | ✅ |
| `DELETE /api/v1/seller/products/{id}` | ❌ | ✅ (own products only) | ❌ | ✅ |
| `POST /api/v1/cart/` | ✅ (guest) | ✅ | ✅ | ❌ |
| `POST /api/v1/checkout/initiate` | ✅ (buyer + guest) | ❌ (self-purchase block BR-012) | ❌ | ❌ |
| `GET /api/v1/orders/` | ✅ (own orders) | ❌ | ❌ | ✅ |
| `GET /api/v1/seller/orders/` | ❌ | ✅ (own store orders) | ❌ | ✅ |
| `PUT /api/v1/seller/orders/{id}/status` | ❌ | ✅ (own store orders) | ❌ | ✅ |
| `GET /api/v1/seller/analytics/` | ❌ | ✅ (own store data) | ❌ | ✅ |
| `GET /api/v1/admin/sellers/` | ❌ | ❌ | ❌ | ✅ |
| `PUT /api/v1/admin/sellers/{id}/approve` | ❌ | ❌ | ❌ | ✅ |
| `GET /api/v1/admin/payouts/` | ❌ | ❌ | ❌ | ✅ |
| `POST /api/v1/analytics/track` | ✅ | ✅ | ✅ | ✅ |

### 4.4 Multi-Tenant Data Isolation

**Rule:** Every seller-scoped ORM query must include `seller=request.user.seller_profile` (or equivalent `seller_id` filter). No exceptions.

**Implementation pattern:**
```python
# ✅ Correct — always scoped
products = Product.objects.filter(
    seller=request.user.seller_profile,
    status='ACTIVE'
)

# ❌ Incorrect — returns all sellers' products
products = Product.objects.filter(status='ACTIVE')
```

**Enforcement:**
- Code review checklist item: "Does every seller-scoped queryset include a seller_id filter?"
- Phase 9 security tests include IDOR test cases: attempt to access Seller A's products while authenticated as Seller B — must return 404 (not 403, to avoid confirming existence)

### 4.5 Subscription Gate

Seller endpoints gated by `IsSellerWithActiveSubscription` check both:
1. `request.user.role == 'SELLER'`
2. `request.user.seller_profile.subscription.status == 'ACTIVE'`

Both conditions checked from JWT claims (role, seller_status) — no DB round-trip per request.

---

## 5. Data Classification & Encryption

### 5.1 Data Classification

| Classification | Description | Examples | Handling |
|---------------|-------------|----------|---------|
| **Public** | No restriction | Product names, prices, images, store descriptions | Cacheable; CloudFront-served |
| **Internal** | Business operational, not PII | Order IDs, analytics aggregate stats, platform metrics | Authenticated access only |
| **Confidential** | PII or sensitive business data | Buyer email/phone/address, seller name/GSTIN, order details | Authenticated + role-scoped access; encrypted at rest |
| **Critical** | Financial or credential data | Bank account numbers, JWT signing key, Razorpay secrets | Field-level encryption + Secrets Manager; audit logged |

### 5.2 Encryption at Rest

| Layer | Mechanism | Coverage |
|-------|-----------|---------|
| RDS PostgreSQL | AWS KMS AES-256 (storage encryption) | All database data at storage level |
| S3 (images, invoices) | AWS S3 SSE-S3 (AES-256) | All objects in shopnest-media bucket |
| ElastiCache Redis | In-transit TLS (at-rest encryption optional — enabled) | All cached data |
| `PayoutDetails.account_number` | Application-layer AES-256 (PyCA `cryptography` library) | Bank account number only |
| ECS task ephemeral storage | Not encrypted — secrets never written to disk | N/A |

**Field-level encryption detail (`PayoutDetails.account_number`):**
```python
# Key: 256-bit AES key stored in AWS Secrets Manager
# Encryption: AES-256-GCM (authenticated encryption)
# Implementation: cryptography.hazmat.primitives.ciphers.aead.AESGCM
# Key rotation: annually or on suspected compromise
# Storage format: base64(iv + ciphertext + tag) in VARCHAR(500) column
```

### 5.3 Encryption in Transit

| Connection | Protocol | Certificate |
|------------|----------|------------|
| Client → ALB | TLS 1.2+ (TLS 1.3 preferred) | AWS ACM auto-renewed cert for `shopnest.in` and `api.shopnest.in` |
| ALB → ECS (Django/Next.js) | HTTP (within VPC, security group protected) | No cert required — VPC internal |
| ECS → RDS | PostgreSQL TLS (`sslmode=require`) | RDS CA certificate |
| ECS → ElastiCache | Redis TLS (`ssl_cert_reqs=required`) | ElastiCache managed cert |
| ECS → AWS Secrets Manager | HTTPS (AWS SDK) | AWS root CA |
| ECS → AWS SES | HTTPS (AWS SDK) | AWS root CA |
| ECS → Razorpay API | HTTPS | Razorpay certificate |

**HTTPS enforcement:**
ALB HTTP listener (port 80) redirects all traffic to HTTPS (port 443) with HTTP 301. No plain HTTP traffic reaches ECS containers.

---

## 6. Network Security

### 6.1 VPC Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  VPC: 10.0.0.0/16  (ap-south-1)                        │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  PUBLIC SUBNET  10.0.1.0/24 (AZ-a)  10.0.2.0/24 (AZ-b)         │  │
│  │                                                                  │  │
│  │  [Internet Gateway] → [ALB] → [WAF rules applied]               │  │
│  │  [NAT Gateway] (for private subnet outbound)                    │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                           │                                             │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  PRIVATE SUBNET  10.0.3.0/24 (AZ-a)  10.0.4.0/24 (AZ-b)        │  │
│  │                                                                  │  │
│  │  [ECS: shopnest-api] [ECS: shopnest-frontend]                   │  │
│  │  [ECS: shopnest-celery] [ECS: shopnest-celery-beat]             │  │
│  │  Outbound: via NAT Gateway (AWS API calls, Razorpay, SES)       │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                           │                                             │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  ISOLATED SUBNET  10.0.5.0/24 (AZ-a)  10.0.6.0/24 (AZ-b)       │  │
│  │                                                                  │  │
│  │  [RDS PostgreSQL Multi-AZ]   [ElastiCache Redis]                │  │
│  │  No internet access. No NAT route.                               │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Security Group Rules

| Security Group | Inbound | Outbound |
|---------------|---------|---------|
| `sg-alb` | 443 from 0.0.0.0/0 (WAF filtered); 80 from 0.0.0.0/0 (redirect only) | 8000 to sg-app (Django); 3000 to sg-app (Next.js) |
| `sg-app` | 8000 from sg-alb (Django); 3000 from sg-alb (Next.js) | 5432 to sg-data (PostgreSQL); 6379 to sg-data (Redis); 443 to 0.0.0.0/0 via NAT (AWS APIs) |
| `sg-data` | 5432 from sg-app (PostgreSQL); 6379 from sg-app (Redis) | None |

**Rule:** No inbound rule permits direct internet access to any resource in the Private or Isolated subnets. All external traffic enters through the ALB in the public subnet.

### 6.3 AWS WAF Rules (attached to ALB)

| Rule Group | Rule | Action |
|-----------|------|--------|
| `AWSManagedRulesCommonRuleSet` | SQLi protection | Block |
| `AWSManagedRulesCommonRuleSet` | XSS protection | Block |
| `AWSManagedRulesSQLiRuleSet` | Additional SQLi patterns | Block |
| Custom — GeoIP | Allow IN (India) only | Block all non-IN traffic |
| Custom — Rate limit | > 100 req/5min per IP on `/api/v1/auth/*` | Block for 5 minutes |
| Custom — Rate limit | > 1000 req/5min per IP on all routes | Block for 5 minutes |
| `AWSManagedRulesKnownBadInputsRuleSet` | Log4j, Spring4Shell, etc. | Block |

**WAF Logging:** All WAF decisions (Block/Allow) logged to CloudWatch Logs `/shopnest/waf/` for incident analysis.

---

## 7. Secrets Management

### 7.1 Secret Inventory

| Secret | Secrets Manager Key | Rotation Policy | Consumers |
|--------|--------------------|--------------  |-----------|
| RS256 JWT private key | `shopnest/jwt/private_key` | Annual / on compromise | Django API, Celery |
| Razorpay Key ID + Secret | `shopnest/razorpay/credentials` | On credential rotation | Django API |
| Razorpay webhook secret | `shopnest/razorpay/webhook_secret` | On credential rotation | Django API |
| PostgreSQL password | `shopnest/db/password` | Annual | Django API, Celery |
| Redis auth token | `shopnest/redis/auth_token` | Annual | Django API, Celery |
| Field encryption key (AES-256) | `shopnest/encryption/field_key` | Annual / on compromise | Django API |
| AWS SES SMTP credentials | `shopnest/ses/smtp` | Annual | Celery (email tasks) |
| Guest tracking HMAC secret | `shopnest/guest/hmac_secret` | Annual | Django API |

### 7.2 Secret Access Pattern

```
ECS Task Startup:
  1. ECS Task Role (IAM) grants GetSecretValue permission on specific ARNs
  2. Django entrypoint script: secrets fetched from Secrets Manager via boto3
  3. Values injected into Django settings as environment variables
  4. Secrets cached in container memory for lifetime of task
  5. Secrets never written to disk, never logged, never returned in API responses

IAM Least Privilege:
  - shopnest-api Task Role → GetSecretValue on shopnest/jwt/*, shopnest/razorpay/*, shopnest/db/*, shopnest/redis/*, shopnest/encryption/*, shopnest/ses/*, shopnest/guest/*
  - shopnest-celery Task Role → same except no shopnest/razorpay/webhook_secret
  - No wildcard ARN permissions — each secret explicitly listed
```

### 7.3 Key Rotation

**JWT RS256 key rotation procedure:**
1. Generate new RSA-2048 key pair
2. Store new private key in Secrets Manager (versioned — `AWSPENDING` staging label)
3. Update `JWKS_JSON` environment variable to include BOTH old and new public keys (key rotation overlap window = 24h = access token TTL)
4. Deploy new ECS task definition (picks up new private key for signing)
5. After 24h: remove old public key from JWKS endpoint; promote new key to `AWSCURRENT`
6. All existing valid access tokens (signed with old key) expire within 24h naturally

---

## 8. Payment Security (PCI-DSS)

### 8.1 PCI-DSS Scope Reduction

ShopNest operates at **PCI-DSS SAQ A** (the lowest scope level) by delegating all card data handling to Razorpay:

| What ShopNest Does | What Razorpay Does |
|-------------------|-------------------|
| Creates a Razorpay Order via server-to-server API call | Hosts the payment form (Razorpay Checkout.js) |
| Receives `razorpay_payment_id` after payment capture | Processes and stores card data |
| Verifies Razorpay webhook HMAC signature | Issues webhook callbacks |
| Stores `razorpay_payment_id` as reference | Maintains PCI-DSS Level 1 certification |

**ShopNest never receives, processes, or stores:**
- Card numbers (PAN)
- Card verification values (CVV/CVC)
- Cardholder names as payment identifiers
- Magnetic stripe data

**Implementation enforcement (NFR-SEC-007):**
- No card data fields in any Django model
- No card data logging in any CloudWatch log group (log format explicitly excludes payment request bodies)
- `razorpay_payment_id` is a Razorpay-issued opaque reference — not card data

### 8.2 Razorpay Webhook Security

```python
# HMAC-SHA256 signature verification on every webhook
import hmac
import hashlib

def verify_razorpay_webhook(body: bytes, signature: str, secret: str) -> bool:
    expected = hmac.new(
        secret.encode('utf-8'),
        body,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)

# Additional protection: WebhookIdempotencyLog
# Unique constraint on razorpay_webhook_id prevents duplicate processing
# (NFR-REL-005)
```

### 8.3 Subscription Billing Security

- Razorpay Subscriptions API manages recurring billing — ShopNest never handles bank account or card details for subscription payments
- Subscription status synchronized via Razorpay webhooks (`subscription.charged`, `subscription.halted`)
- Subscription `status` field in Django is a read-only mirror of Razorpay state — never modified by seller-accessible API endpoints

---

## 9. Regulatory Compliance

### 9.1 DPDPA 2023 (Digital Personal Data Protection Act)

| Requirement | ShopNest Implementation | Status |
|------------|------------------------|--------|
| Data localization | All PII in AWS ap-south-1 (Mumbai); no cross-border transfers | ✅ Implemented by architecture |
| Consent for data collection | Analytics events: user notified in Privacy Policy; account creation implies consent | ✅ PRD requirement |
| Right to erasure | Account deletion: PII fields anonymized, `user_id` in analytics set to NULL, financial records retained 7 years per BR-011 | ✅ DATA-MODEL.md: deletion policy |
| Data minimisation | Analytics properties: no PII collected beyond user_id; DPDPA flag per property in PRD-ANALYTICS-PLAN.md | ✅ PRD-ANALYTICS-PLAN.md |
| Data retention limits | Analytics events: 2 years; financial records: 7 years; WebhookIdempotencyLog: 90 days | ✅ DATA-MODEL.md |
| Security of personal data | AES-256 at rest (RDS + S3); TLS 1.2+ in transit; access controls | ✅ This document |
| Breach notification | Incident Response procedure triggers within 72 hours of confirmed breach (see §12) | 🔲 Policy — must be documented pre-launch |

### 9.2 Consumer Protection (E-Commerce) Rules 2020

| Requirement | ShopNest Implementation | Status |
|------------|------------------------|--------|
| Seller identification | Store name, GSTIN (optional at MVP), contact details displayed | ✅ PRD FR-STORE-001 |
| Grievance Officer | Admin contact displayed in footer | 🔲 OQ-005 — legal review open |
| Country of origin | Product detail page field (MVP: self-declared by seller) | 🔲 Post-MVP field — not in MVP scope |
| Price disclosure | Total price (including all taxes) shown at checkout | ✅ PRD FR-CHECKOUT-001 |
| Return/cancellation policy | Cancellation before confirmation only (no returns MVP) | ✅ REQUIREMENTS.md BR-010 |
| Invoice | GST invoice generated via WeasyPrint and stored in S3 | ✅ TECH-STACK.md |

### 9.3 IT Act 2000 / IT (Amendment) Act 2008

- ShopNest operates as an "intermediary" under Section 79 — liability protection requires take-down mechanism for unlawful content (product listings)
- Admin product approval workflow (FR-PROD-009) and product rejection capability satisfy intermediary due diligence obligations
- User data disclosure to government authorities: governed by Section 69 court orders — no automatic sharing

---

## 10. OWASP Top 10 (2021) Mitigation

| # | Vulnerability | Risk Level | ShopNest Control | Verification |
|---|--------------|-----------|-----------------|-------------|
| A01 | Broken Access Control | Critical | DRF Permission Classes; seller_id ORM filtering; `IsOwnerOrAdmin` check; 404 on cross-tenant IDOR | Phase 9: IDOR test suite; Phase 10: ZAP DAST |
| A02 | Cryptographic Failures | High | TLS 1.2+ (ALB); AES-256 at rest (KMS); RS256 JWT (not HS256); BCrypt cost 12; AES-256-GCM field encryption for bank account numbers | Phase 10: TLS scan; certificate pinning review |
| A03 | Injection | Critical | Django ORM parameterised queries (no raw SQL except search_vector trigger); DRF serializers validate all input; AWS WAF SQLi managed rules | Phase 9: SQL injection tests; Phase 10: ZAP active scan |
| A04 | Insecure Design | High | Threat model documented (§2); STRIDE analysis per domain; security requirements as NFRs in REQUIREMENTS.md | Architecture review at Human Gate |
| A05 | Security Misconfiguration | High | Django `DEBUG=False` in production; SECRET_KEY from Secrets Manager (not settings file); `ALLOWED_HOSTS` restricted; CloudWatch alarms on security events | Phase 10: configuration audit |
| A06 | Vulnerable Components | High | ECR image scanning on push; `pip-audit` in CI/CD pipeline (GitHub Actions); Dependabot alerts on GitHub repo | Phase 11: CI/CD security scan gate |
| A07 | Identification & Authentication Failures | Critical | BCrypt cost 12; RS256 JWT (24h expiry); refresh token denylist; HttpOnly + Secure + SameSite=Strict cookie; WAF rate limit on auth endpoints | Phase 9: auth tests; Phase 10: credential stuffing simulation |
| A08 | Software and Data Integrity Failures | High | Razorpay HMAC-SHA256 webhook verification; WebhookIdempotencyLog unique constraint; Docker image digest pinning in ECS task definitions; `pip install --require-hashes` in CI | Phase 9: webhook replay test; Phase 11: image digest verification |
| A09 | Security Logging and Monitoring Failures | High | AuditLog table (all state changes with before/after snapshots); CloudWatch structured logs; WAF logging; CloudWatch alarms on 4xx/5xx spikes; Phase 13 monitoring | Phase 13: Monitoring setup |
| A10 | Server-Side Request Forgery (SSRF) | Medium | No server-side URL fetch from user-supplied URLs; Razorpay Checkout.js loaded from Razorpay CDN (not proxied); S3 image upload via pre-signed URL (client-direct, no server proxy) | Phase 10: SSRF test cases |

---

## 11. Security Testing Requirements

### 11.1 Phase 9 (Testing) — Security Test Cases

| Test ID | Test | Expected Result |
|---------|------|----------------|
| SEC-001 | IDOR: Seller A attempts GET /api/v1/seller/products/{seller_B_product_id} | 404 (not 403) |
| SEC-002 | IDOR: Seller A attempts GET /api/v1/seller/orders/{seller_B_order_id} | 404 |
| SEC-003 | IDOR: Buyer attempts GET /api/v1/seller/analytics/ | 403 |
| SEC-004 | Expired access token on protected endpoint | 401 |
| SEC-005 | Tampered JWT (role: "ADMIN" forged in HS256) | 401 (RS256 signature verification fails) |
| SEC-006 | Razorpay webhook replay (duplicate webhook_id) | 200 (idempotent — no duplicate processing) |
| SEC-007 | Razorpay webhook with invalid HMAC signature | 400 |
| SEC-008 | SQL injection in product search query (`?q=' OR 1=1--`) | 400 (ORM parameterised query; no SQL injection) |
| SEC-009 | XSS payload in product name via seller API | 400 (DRF serializer validation rejects HTML) |
| SEC-010 | Self-purchase: seller attempts checkout on own product | 403 (BR-012: self-purchase block) |
| SEC-011 | Stock decrement race condition (concurrent checkout, 1 unit stock) | One order succeeds, others get 409 (stock insufficient) |
| SEC-012 | Logout: refresh token used after logout | 401 (denylist check) |
| SEC-013 | Admin endpoint accessed by SELLER role | 403 |
| SEC-014 | Unauthenticated access to seller dashboard endpoint | 401 |
| SEC-015 | Seller with SUSPENDED subscription attempts product creation | 403 |

### 11.2 Phase 10 (Security) — External Security Review

- OWASP ZAP DAST scan against staging environment
- TLS configuration scan (verify TLS 1.2+ minimum, no SSLv3/TLS 1.0)
- Dependency vulnerability scan (`pip-audit`, `npm audit`)
- Docker image CVE scan review (ECR scan results)
- Manual penetration test: authentication flows, IDOR patterns, payment manipulation

---

## 12. Incident Response

### 12.1 Severity Classification

| Severity | Definition | Examples | Response SLA |
|----------|-----------|---------|-------------|
| P1 — Critical | Active data breach or system compromise | Database exfiltration; JWT key compromise; Razorpay secret exposure | 1 hour (immediate) |
| P2 — High | Potential breach or major security control failure | WAF disabled; unusual auth failure spike; suspicious admin activity | 4 hours |
| P3 — Medium | Security misconfiguration or vulnerability discovered | Dependency CVE; misconfigured S3 bucket | 24 hours |
| P4 — Low | Security improvement opportunity | Outdated TLS cipher; missing security header | Next sprint |

### 12.2 P1 Response Procedure

1. **Contain:** Immediately disable affected service (ECS task count → 0 if needed); revoke compromised credentials in Secrets Manager; block suspected IPs in WAF
2. **Assess:** Review CloudWatch logs and AuditLog table to determine breach scope
3. **Notify:** Internal team (< 1 hour); DPDPA 72-hour notification obligation to Data Protection Board of India if PII breach confirmed
4. **Remediate:** Rotate all secrets; deploy patched build; restore from RDS PITR if data corruption
5. **Review:** Post-incident review within 5 business days; update threat model and controls as needed

### 12.3 Monitoring Triggers (Phase 13)

| Alert | Threshold | Action |
|-------|-----------|--------|
| Auth failure rate | > 20 failures/minute from single IP | WAF block; P2 alert |
| 5xx error rate | > 5% of requests over 5-minute window | P2 alert; ECS health check review |
| JWT validation failure spike | > 50 invalid JWTs/minute | P1 investigation |
| WAF block rate | > 1,000 blocks/hour | P2 alert; review WAF logs |
| RDS failed login | Any | P1 immediate alert |
| Secrets Manager access from unexpected principal | Any | P1 immediate alert |

---

## References

- REQUIREMENTS.md: All NFR-SEC-* requirements (NFR-SEC-001 through NFR-SEC-010)
- ADR-004: RS256 JWT Authentication Strategy
- ADR-005: AWS ECS Fargate Deployment Platform (VPC, WAF, network topology)
- ADR-003: PostgreSQL 16 (encryption at rest, PITR)
- DATA-MODEL.md: AuditLog, WebhookIdempotencyLog, analytics_events retention
- API-SPEC.md: Auth endpoint specifications, webhook verification
- OWASP Top 10 2021: https://owasp.org/Top10/
- DPDPA 2023: Digital Personal Data Protection Act, India
- Razorpay Security Documentation: https://razorpay.com/docs/payments/security/
