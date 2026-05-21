# Definition of Done — ShopNest
**Phase:** 06 — Task Breakdown & Sprint Planning
**Generated:** 2026-05-19
**Authority:** This DoD is the binding agreement between the engineering team, QA, and the product owner. A task is either Done (all criteria met) or Not Done (one or more criteria unmet). There is no "mostly done."

---

## Universal DoD — All Tasks

A task is **DONE** when ALL of the following are true:

### 1. Code Quality

- [ ] Code is self-reviewed by the author before opening a PR (author has read every line they wrote)
- [ ] `black --check backend/` passes with zero formatting errors (backend)
- [ ] `isort --check-only backend/` passes with zero import order errors (backend)
- [ ] `npm run lint` passes with zero ESLint errors (frontend)
- [ ] `mypy backend/` passes with zero type errors on the changed files (backend)
- [ ] `npm run type-check` passes with zero TypeScript errors (frontend)
- [ ] No `# noqa`, `// eslint-disable`, or `@ts-ignore` added without a code comment explaining the exact reason
- [ ] No secrets, API keys, passwords, or PII in committed code (verified by author self-review)

### 2. Testing

- [ ] Unit tests written for all new business logic (backend: pytest; frontend: Jest + React Testing Library)
- [ ] `pytest --cov=. --cov-report=term-missing` reports ≥ 80% line coverage for the modified Django app(s)
- [ ] `npm run test -- --coverage` reports ≥ 80% line coverage for the modified Next.js module(s)
- [ ] Integration tests written for all new API endpoints (at minimum: happy path, validation failure, auth failure)
- [ ] No test uses `pytest.mark.skip`, `test.skip()`, or `test.todo()` without a linked GitHub issue explaining why

### 3. Code Review

- [ ] PR opened against `develop` branch (never directly against `main`)
- [ ] PR description contains: what changed, why, how to test it manually
- [ ] At least 1 peer approval received (any other team member)
- [ ] All review comments resolved or explicitly acknowledged with a reply before merge
- [ ] GitHub Actions CI pipeline is green (all steps pass) on the PR head commit

### 4. API Contract (Backend Tasks Only)

- [ ] Response schema matches the contract defined in `docs/design/API-SPEC.md`
- [ ] HTTP status codes are correct per the spec (e.g., 201 for creation, 409 for conflict, not 500 for expected errors)
- [ ] All snake_case JSON fields (no camelCase)
- [ ] `@extend_schema` decorator added to the DRF view so `drf-spectacular` generates correct OpenAPI docs
- [ ] `python manage.py spectacular --validate` still passes after the new endpoint is added

### 5. Security (Non-Negotiable)

- [ ] No raw card numbers, CVVs, expiry dates, or Razorpay secrets appear in code, logs, or API responses
- [ ] No plaintext passwords in code, logs, or API responses
- [ ] Bank account numbers masked to last 4 digits in all API responses
- [ ] All new endpoints with authentication checked against the RBAC matrix (seller endpoints have `IsSeller`; admin endpoints have `IsAdmin`)
- [ ] File uploads validated server-side: MIME type (JPEG/PNG only), size limits enforced
- [ ] No user-supplied input passed to shell commands, raw SQL, or `eval()`

### 6. TASKS.md Updated

- [ ] `docs/planning/TASKS.md` updated: task status changed from `Pending` to `Done`
- [ ] If any acceptance criterion required clarification during implementation, it is updated in TASKS.md to reflect the actual implemented behavior

---

## Additional DoD — Frontend Tasks Only

- [ ] Screen matches the wireframe specification in the referenced `docs/visuals/ux/SCR-NNN-*.html` file for all documented states
- [ ] Screen is functional at 375px viewport width (buyer-facing screens)
- [ ] Screen is functional at 768px viewport width (seller dashboard screens)
- [ ] Screen is functional at 1280px viewport width (admin portal screens)
- [ ] Loading skeleton or spinner shown during data fetch (no raw blank white screen)
- [ ] Empty state rendered when API returns no data (no blank section)
- [ ] Error state rendered when API call fails (inline error message — no unhandled exception)
- [ ] All interactive elements (buttons, links, form inputs) reachable via keyboard Tab navigation
- [ ] All `<img>` tags have meaningful `alt` text (not empty, not "image")
- [ ] All form inputs have an associated `<label>` element (either `<label for>` or `aria-label`)

---

## Additional DoD — Database Tasks Only

- [ ] Migration file generated via `python manage.py makemigrations` (not hand-written)
- [ ] Migration applies cleanly on a fresh database: `python manage.py migrate` exits 0
- [ ] Migration is reversible: `python manage.py migrate <app> zero` exits 0 (if rollback is feasible)
- [ ] All new indexes documented in `docs/design/DATA-MODEL.md` with justification

---

## Additional DoD — DevOps Tasks Only

- [ ] Infrastructure change documented in `infra/README.md` or the relevant workflow file
- [ ] No hardcoded credentials or environment-specific values in CI/CD workflows (all use GitHub Secrets or env vars)
- [ ] Pipeline change tested on a branch before merging to `develop`

---

## Additional DoD — Celery / Async Tasks Only

- [ ] Task is registered in the correct Celery queue (`default`, `email`, `analytics`, or `payouts`)
- [ ] Task is idempotent (calling it twice with the same arguments produces the same result as calling it once)
- [ ] Retry logic implemented: `autoretry_for=(Exception,)`, `max_retries=3`, `countdown=30` (exponential where appropriate)
- [ ] Failure after all retries logs to structured JSON log with: `event_type`, `entity_id`, error message
- [ ] Task does NOT raise an exception to the caller — failures are logged and handled gracefully

---

## Performance Benchmarks (Required Before Sprint 8 Merge)

These are enforced at Sprint 8 quality gate, not on every task:

- [ ] P95 API response time < 200ms measured on staging with 100 concurrent users (CloudWatch or k6)
- [ ] P95 search response time < 500ms against 50,000 product fixture
- [ ] Marketplace homepage LCP < 2.5s on mobile 4G simulation (Google PageSpeed Insights)
- [ ] Product image load time < 1.0s via CloudFront CDN (image ≤ 500KB)

---

## Definition of Ready (Before a Task Enters "In Progress")

A task is **READY** to be started when:

- [ ] The task has a clear title, description, and at least 3 acceptance criteria in TASKS.md
- [ ] All tasks it depends on are Done (or will not be needed for this task to begin)
- [ ] The assignee has read and understood the wireframe reference (for frontend tasks)
- [ ] The assignee has read the relevant section of API-SPEC.md (for backend tasks)
- [ ] No Tier 1 open questions block this task's implementation decisions

---

## Sprint Done Criteria (Sprint-Level Gate)

A sprint is **DONE** when ALL of the following are true:

- [ ] All committed tasks are individually Done per this DoD
- [ ] No P0 bugs introduced in this sprint remain open (P0 = functionality regression or security issue)
- [ ] Sprint retrospective conducted (even 30-minute async format)
- [ ] Sprint velocity (actual SP completed) recorded for velocity recalibration
- [ ] SPRINT-PLAN.md updated with actual completed tasks and any tasks carried forward
- [ ] Staging environment is deployable and stable at end of sprint

---

*This DoD applies to all phases of ShopNest MVP development (Phase 7 implementation through Phase 9 testing).*
*Changes to this DoD require agreement from all 3 team members and a version note added below.*

**Version History:**
| Version | Date | Change | Agreed By |
|---------|------|--------|-----------|
| 1.0 | 2026-05-19 | Initial DoD — Phase 6 task breakdown | Phase 6 Agent |
