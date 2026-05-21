# Task Breakdown — Assumption Log
**Phase:** 06 — Task Breakdown & Sprint Planning
**Agent:** Phase 6 Task Breakdown Agent
**Generated:** 2026-05-19
**Session:** Full task backlog, sprint plan, dependency map, and definition of done for ShopNest MVP

---

## Tier 3 Inferences Made This Phase

| ID | Inference | Basis | Confidence | Must Validate Before Phase |
|----|-----------|-------|------------|--------------------------|
| A-06-001 | 6 productive hours per developer per day (after meetings, context switching, code review overhead, and standups subtracted from 8h) | Mike Cohn — Agile Estimating and Planning (2005); industry standard for knowledge work capacity planning | High | Phase 7 Sprint 1 retrospective — adjust if actuals differ |
| A-06-002 | 1 story point = 4 developer-hours for this team and stack | Inferred from Cohn's SP calibration guidance applied to this team composition; S tasks (8h) assigned 2 SP throughout | High | Phase 7 Sprint 2 retrospective — recalibrate if velocity deviates ≥ 20% |
| A-06-003 | Sprint 1 ramp-up discount of 20% applied (reducing committable SP from 28 to 22) | Empirical finding in Cohn (2005) and Schwaber & Sutherland Scrum Guide — first sprint of a new greenfield project involves tooling setup, context loading, and team calibration | High | Validated at Sprint 1 retrospective |
| A-06-004 | Code review overhead budgeted at 30 minutes per task (averaged across task sizes) | Industry benchmark for small team PR review cycles [Industry benchmark — not project-specific. Confirm with team.] | Medium | Sprint 1 actual review time log |
| A-06-005 | Gateway fee for Razorpay computed as exactly 2% flat for MVP ledger calculations | CLAUDE.md Architecture Decisions: "Razorpay payment gateway fee (~2%+GST) is absorbed by ShopNest"; fee formula `floor(gross_amount_paise × 0.02)` chosen for integer paise consistency | High | Phase 7 — confirm exact Razorpay fee schedule before TASK-048 implementation; actual fee may vary by payment method |
| A-06-006 | Monday 09:00 IST = Monday 03:30 UTC for Celery Beat cron expression: `crontab(hour=3, minute=30, day_of_week=1)` | IST = UTC+5:30; IST 09:00 − 5:30 = UTC 03:30; standard timezone arithmetic | High | Phase 7 TASK-049 — verify in staging environment before production deploy |
| A-06-007 | Subscription price ₹1,999/month stored as environment variable `SUBSCRIPTION_PRICE_PAISE=199900` (1 INR = 100 paise) | CLAUDE.md OQ-003 resolution: "Hardcode for MVP, flag as env var"; paise convention from CLAUDE.md Architecture Decisions | High | Phase 7 — OQ-003 still open; confirm price with product owner before first seller signs up |
| A-06-008 | Bank account update mid-payout-cycle policy: existing cycle runs to old account; new account takes effect next Monday. Implemented in TASK-055 | CLAUDE.md Architecture Decisions: "OQ-006 resolution: implement: hold existing payout, process to old details" | High | Phase 7 TASK-055 — confirm with business stakeholder before implementation |
| A-06-009 | Admin portal is desktop-only (minimum 1280px) with no mobile requirement | UX WIREFRAMES.md §2a: "Admin = desktop-only"; NFR-USA-002 sets admin minimum at 768px but UX design targets 1280px; no admin mobile wireframes produced in Phase 5 | High | Phase 7 — acceptable for MVP; post-MVP enhancement if ops team is mobile-first |
| A-06-010 | Razorpay gateway fee is treated as an operating cost absorbed by ShopNest (not passed to sellers); `gateway_fee_paise` tracked in SettlementLedgerEntry for internal accounting | CLAUDE.md Architecture Decisions C-001 resolution: "sellers receive order amount net of gateway fee"; DATA-MODEL.md: SettlementLedgerEntry has both `gross_amount_paise` and `gateway_fee_paise` fields | High | Phase 7 — validate Razorpay fee deduction before payout logic finalized |
| A-06-011 | WeasyPrint PDF generation will work on the Docker image base (python:3.12-slim-bullseye or similar Alpine-adjacent). Native dependencies (Pango, Cairo, libffi) may require a Dockerfile adjustment. | WeasyPrint 60.x documentation notes native library requirements; Alpine vs. Debian distinction matters | Medium | Phase 7 TASK-037 — prototype Docker build with WeasyPrint dependencies before Sprint 5 |
| A-06-012 | Redis product catalog cache TTL set at 1 hour for GET /api/v1/products/ and search responses, invalidated on Product model save signal | ARCHITECTURE.md: "Redis product catalog cache (TTL 1h)"; ADR-007 pattern | High | Phase 7 TASK-070 — validate invalidation logic doesn't cause excessive cache misses during active product approval |
| A-06-013 | `python-magic` library used for server-side MIME type validation (reads file magic bytes, not extension) | SECURITY-ARCHITECTURE.md: "Server-side MIME validation using `python-magic` (reads file magic bytes, not extension)"; standard security pattern for file upload | High | Phase 7 TASK-035, TASK-041 — verify `python-magic` + `libmagic` available in Docker base image |
| A-06-014 | Total task count is 84 tasks (TASK-001 through TASK-084) with 84 unique IDs; no tasks are XL (> 3 days) — all L tasks are 5 SP (20h = 2.5 dev-days) which is within the 1-week atomic rule | Bill Wake INVEST criteria: tasks should be completable by one developer in a predictable timeframe; 5 SP = ~20h is within a 2.5-day window for one developer | High | Phase 7 — flag if any L task actually takes > 5 working days and split at that point |
| A-06-015 | Sprint 8 extended to 3 weeks (15 working days) from the standard 2 weeks to accommodate ECS deployment setup, load testing against a live staging environment, and accessibility audit requiring a deployed build | Sprint 8 tasks (TASK-073 load test, TASK-072 accessibility audit) require a functioning staging environment that itself requires TASK-073 (ECS CD pipeline) to be complete first — this circular dependency necessitates extended time | Medium | Phase 7 Sprint 8 planning — confirm stakeholder acceptance of 3-week Sprint 8 at Sprint 7 retrospective |
| A-06-016 | AuditLog entries are written asynchronously via Celery to the `default` queue to avoid blocking the critical path on every money-affecting event | ARCHITECTURE.md ADR-001: "Signal-Driven Audit"; ADR-006: "async task processing for non-blocking operations" | High | Phase 7 TASK-084 — verify AuditLog Celery task is truly fire-and-forget and failures do not propagate to the user |
| A-06-017 | `docs/planning/` directory created as a new directory (did not previously exist); no existing files were overwritten | File system state at phase execution start; Glob scan showed no prior planning/ directory | High | N/A — confirmed by file creation success |

---

## Open Flags (Tier 2 — Unconfirmed Suggestions)

| Flag ID | Suggestion Made | Location in Artifact | Status |
|---------|----------------|---------------------|--------|
| F-06-001 | Sprint 5 backend dev load (25 SP in 10 days) is at the upper boundary of sustainable pace. TASK-029, TASK-036, TASK-037, TASK-038, TASK-042, TASK-045, TASK-047, TASK-078, TASK-079 total 25 SP for the backend dev. Suggest moving TASK-078 or TASK-079 to Sprint 6 if sprint 5 day-5 checkpoint shows > 50% of committed SP still pending | SPRINT-PLAN.md Sprint 5 | Pending confirmation at Sprint 4 retrospective |
| F-06-002 | TASK-044 (product form UI, 3 SP) floated between Sprint 5 and Sprint 6 in the plan. Assign definitively to Sprint 5 if seller product management UI (TASK-043) completes by Sprint 5 day 7; otherwise move to Sprint 6 | SPRINT-PLAN.md Sprint 5/6 boundary | Pending — decide at Sprint 5 day 7 checkpoint |
| F-06-003 | Load test (TASK-076) requires a functioning ECS staging environment. If TASK-073 ECS pipeline encounters unexpected AWS configuration blockers, load test may need to run against a local `docker compose` scale-out instead. This is an acceptable MVP fallback but should be flagged to stakeholders. | SPRINT-PLAN.md Sprint 8, DEPENDENCY-MAP.md §5 | Pending — confirm AWS account ECS/ECR access before Sprint 8 |

---

## Resolution Log

| ID | Original Assumption | Resolution | Resolved By | Date |
|----|--------------------|-----------|-----------:|------|
| — | No assumptions have been resolved yet — this log will be updated during Phase 7 implementation as inferences are validated or corrected | — | — | — |

---

*Generated by Phase 6 Task Breakdown Agent on 2026-05-19*
*All Tier 3 inferences are logged with their source. High-confidence items proceed to implementation; Medium-confidence items require validation spike or prototype before full implementation.*
