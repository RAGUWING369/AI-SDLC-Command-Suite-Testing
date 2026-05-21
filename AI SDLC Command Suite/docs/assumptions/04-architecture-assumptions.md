# Architecture — Assumption Log

**Phase:** 04 — System Architecture & Design
**Agent:** 04_architecture_agent.md
**Generated:** 2026-05-14
**Session:** ShopNest Phase 4 — Modular Django Monolith architecture design

---

## Tier 3 Inferences Made This Phase

| ID | Inference | Basis | Confidence | Must Validate Before Phase |
|----|-----------|-------|------------|---------------------------|
| A-04-001 | Gunicorn 4 workers × 4 threads = 16 concurrent slots per ECS task is sufficient for 100 req/s steady-state load with 2 API tasks | Industry standard Gunicorn sizing formula; Django ORM is I/O-bound so threaded model appropriate; 200ms avg latency × 100 req/s = 20 concurrent slots needed | High | Phase 9 (load test validation) |
| A-04-002 | PostgreSQL db.t3.small (2 vCPU, 2GB RAM) is sufficient for MVP scale (≤100 sellers, ≤50K products, ≤1K concurrent users) | AWS RDS db.t3.small specifications; PostgreSQL max_connections formula; 2-10 ECS tasks × 10 connections/worker = 100 connections max, within db.t3.small capacity of ~200 | High | Phase 9 (DB load test) |
| A-04-003 | ElastiCache cache.t3.micro (1 vCPU, 0.5GB RAM) is sufficient for Redis broker + cart cache + product cache at MVP scale | MVP data volume: 100 sellers × 500 products = 50K products; cart sessions per 1K users × 24h TTL; Celery task queue backlog; total estimated Redis memory < 200MB | High | Phase 9 (Redis memory monitoring) |
| A-04-004 | PostgreSQL GIN-indexed tsvector search returns P95 < 100ms at 50,000 product records on db.t3.small | PostgreSQL FTS benchmark data for GIN index on tsvector at 50K rows on comparable hardware; includes headroom to 500ms NFR-PE-005 target | High | Phase 9 (search load test with 50K seeded products) |
| A-04-005 | ECS Fargate cold start of ~10 seconds occurs when scaling from 0 to 1 task; maintaining min 2 tasks eliminates this as a user-facing concern | AWS ECS Fargate documented cold start behavior; industry benchmark for Python Django container cold start | High | Phase 13 (ECS task startup time monitoring) |
| A-04-006 | Total AWS monthly cost at MVP scale (100 sellers, 1K users, min-capacity ECS) is approximately $220/month, within CON-001 ($2,000/month) | AWS pricing calculator estimates for: ECS Fargate (3 services at minimum capacity), RDS db.t3.small Multi-AZ, ElastiCache cache.t3.micro, ALB, WAF, CloudFront, S3, SES, Secrets Manager, CloudWatch, ECR | Medium | Phase 12 (actual first month AWS bill review) |
| A-04-007 | Razorpay Payouts API successfully processes batch of 100 sellers in Monday 09:00 IST weekly Celery job within typical batch duration | Razorpay Payouts API documented throughput; 100 sequential API calls at ~200ms each = ~20 seconds for full batch; well within no execution limit | High | Phase 9 (payout task integration test) |
| A-04-008 | BCrypt cost factor 12 adds approximately 250ms per password hash operation, which is acceptable for login throughput at MVP scale | BCrypt cost factor benchmarks on modern hardware; actual login rate at 100 sellers + 1K buyers is well below 1 req/s; 250ms is imperceptible in a login flow | High | Phase 9 (auth load test) |
| A-04-009 | AWS CloudFront CDN serves product images in < 1 second for Indian users from Mumbai, Delhi, and Chennai edge locations | CloudFront edge location proximity to ap-south-1; ≤500KB image size constraint in NFR-PE-004; CDN TTFB benchmarks from Indian edge locations | High | Phase 9 (image load time test from Indian IPs) |
| A-04-010 | SES email delivery rate of ≥95% for transactional emails (order confirmation, payout notification) within 30 seconds of trigger | AWS SES documented delivery SLA; transactional email category (not bulk/marketing) has higher delivery priority; NFR-REL-006 target is 30 seconds | Medium | Phase 9 (email delivery integration test) |
| A-04-011 | RS256 RSA-2048 public key verification latency is negligible (< 1ms) per API request, contributing no measurable overhead to P95 < 200ms target | RSA-2048 public key verification is a CPU-bound operation of ~0.3ms on modern hardware; private key signing at token issuance is ~3ms (login path only, not per-request) | High | Phase 9 (auth overhead measurement in load test) |
| A-04-012 | Django ORM seller_id FK filter on all seller-scoped queries adds < 1ms overhead (B-tree index lookup) vs. unfiltered queries | PostgreSQL B-tree index lookup on UUID FK column is O(log n) — at 50K products, this is deterministically fast; already accounted for in P95 < 50ms NFR-PE-008 DB query target | High | Phase 9 (query EXPLAIN ANALYZE review) |
| A-04-013 | ap-south-1 has at least two availability zones (ap-south-1a and ap-south-1b) available for Multi-AZ ECS and RDS deployment | AWS ap-south-1 officially has 3 AZs (ap-south-1a, ap-south-1b, ap-south-1c); Multi-AZ uses 1a and 1b | High | N/A — infrastructure fact |
| A-04-014 | POST /api/v1/analytics/track can return 202 in < 50ms when the Celery enqueue operation (Redis LPUSH) is the only synchronous operation | Redis LPUSH operation latency from ECS private subnet to ElastiCache in same VPC is < 2ms; total 202 path: JSON parse + schema validate + LPUSH = ~10-20ms P95 | High | Phase 9 (analytics endpoint latency test) |
| A-04-015 | WeasyPrint can generate a GST invoice PDF within 2 seconds (acceptable for background Celery task, not on critical path) | WeasyPrint PDF generation for simple invoice HTML template benchmarks at 0.5-2s on comparable hardware; runs asynchronously via Celery (not blocking checkout) | Medium | Phase 9 (invoice generation time test) |

---

## Open Flags (Tier 2 — Unconfirmed Suggestions)

| Flag ID | Suggestion Made | Location in Artifact | Status |
|---------|----------------|----------------------|--------|
| F-04-001 | Suggested Celery Beat run as separate ECS service (desiredCount=1) to prevent duplicate scheduled job execution | ADR-006: Celery Beat container | Confirmed — user confirmed "No Changes needed" to architecture approach in Phase 4 pre-design confirmation |
| F-04-002 | Suggested Redis DB index separation: db=0 for cart/product cache, db=1 for Celery broker, db=2 for refresh token denylist | ARCHITECTURE.md: Redis usage; ADR-006 | Pending explicit validation — logical Redis DB separation assumed sufficient at MVP scale |
| F-04-003 | Suggested PostgreSQL FTS upgrade trigger: evaluate Elasticsearch when P95 search > 400ms in Phase 9 load test or when product count exceeds 200K | ADR-007: performance validation gate | Accepted as architectural guidance; validation in Phase 9 |
| F-04-004 | Suggested analytics events retain 2 years then purge via monthly Celery Beat task | SECURITY-ARCHITECTURE.md §9.1 DPDPA retention; DATA-MODEL.md | Pending DPDPA legal confirmation — 2 years is a reasonable inference from DPDPA data minimisation principle; exact retention period should be confirmed by legal counsel |

---

## Conflict Resolution Log

| Conflict ID | Description | Resolution | Resolved In |
|-------------|-------------|------------|-------------|
| C-04-001 | Whether to use a single ALB or separate ALBs for Django API and Next.js frontend | Single ALB with path routing (`/*` → Next.js, `/api/v1/*` → Django) — saves ~$20/month vs. 2 ALBs; routing is deterministic on path prefix | ADR-005: ALB path routing rationale |
| C-04-002 | Whether Celery Beat should share the worker container or run separately | Separate ECS service (desiredCount=1) — prevents duplicate job scheduling when worker scales horizontally; horizontal scaling of Beat would trigger duplicate payout jobs | ADR-006: Celery Beat container rationale |

---

## Resolution Log

| ID | Original Assumption | Resolution | Resolved By | Date |
|----|--------------------|-----------|-----------:|------|
| A-04-001 | Gunicorn sizing sufficient for load | To be validated | Phase 9 load test | TBD |
| A-04-002 | db.t3.small sufficient | To be validated | Phase 9 load test | TBD |
| A-04-006 | ~$220/month AWS estimate | To be confirmed against first AWS bill | Phase 12 | TBD |
| F-04-004 | 2-year analytics retention | Requires DPDPA legal confirmation | Legal review (pre-launch) | TBD |
