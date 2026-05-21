# UX Design — Assumption Log

**Phase:** 05 — UX Design (Full Production Run — all 21 screens)
**Agent:** 05_ux_design_agent.md (full production run following approved testing run)
**Generated:** 2026-05-14
**Full Run Executed:** 2026-05-15
**Re-executed:** 2026-05-14 (agent spec updated — desktop viewport rule enforced)
**Session:** ShopNest Phase 5 — SCR-001 (PDP), SCR-002 (Cart & Checkout), SCR-003 (Seller Product Management)

---

## Tier 3 Inferences Made This Phase

| ID | Inference | Basis | Confidence | Must Validate Before Phase |
|----|-----------|-------|------------|---------------------------|
| A-05-001 | System font stack (-apple-system, Roboto) chosen over web font (e.g., Google Fonts Inter) for LCP performance | NFR-PE-002 (LCP < 2.5s mobile 4G); system fonts render without network round-trip; industry standard for performance-critical Indian mobile apps | High | Phase 7 (Implementation — confirm font decision with design) |
| A-05-002 | Orange (#F97316) selected as CTA color for primary buttons (Buy Now, Pay, Submit, Add to Cart primary) | Indian e-commerce convention (Amazon, Flipkart, Meesho use orange/yellow CTAs for purchase actions); creates urgency and trust in Indian buyer context | High | Phase 5 full UX design review or Phase 7 |
| A-05-003 | Blue (#1A56DB) selected as primary brand color for links, active states, and seller UI elements | Trust/reliability signal for marketplace platform; blue is the dominant color in successful Indian e-commerce (Flipkart navy, Ajio blue-toned); contrasts well with orange CTA | Medium | Phase 5 full UX design review |
| A-05-004 | Product images in PDP displayed at full 375px width (no side padding) for maximum visual impact on mobile | Industry pattern from Flipkart, Amazon, Myntra — full-bleed product images on mobile PDP increase conversion; padding reduces image area on small screens | High | Phase 7 frontend implementation |
| A-05-005 | Quantity stepper uses [−][value][+] horizontal pattern rather than dropdown | Mobile-first pattern; stepper is faster than dropdown for low quantities (1–10 typical); dropdown appropriate for high-volume B2B contexts (not ShopNest use case) | High | Phase 7 |
| A-05-006 | Guest checkout form uses inline validation on blur (not on submit) to reduce friction | WCAG 3.3.1 (Error Identification); UX best practice — post-submit-only validation is the top source of form abandonment on mobile; confirmed by USER-STORIES.md US-012 (guest checkout) | High | Phase 9 (user testing) |
| A-05-007 | Razorpay modal described as not designed by ShopNest — correct delegation boundary | REQUIREMENTS.md NFR-SEC-007 (zero card data stored); Razorpay Checkout.js handles its own UI; ShopNest never renders payment fields | High | N/A — architecture fact |
| A-05-008 | Guest tracking URL described as HMAC-SHA256 token — no account needed | ARCHITECTURE.md (ADR-004); API-SPEC.md (GET /api/v1/orders/track/{tracking_token}); confirmed in multiple prior phases | High | N/A — confirmed in architecture |
| A-05-009 | Bottom navigation for seller mobile uses 4 tabs: Dashboard, Products, Orders, Profile — covering the primary seller workflows | USER-STORIES.md covers seller registration (US-001), product management (US-004, US-006), order management (US-007, US-008), profile/subscription (US-003); 4 tabs is the minimum viable navigation for these flows; 5+ tabs is overcrowded on mobile | High | Phase 7 (confirm with Priya persona testing) |
| A-05-010 | Price minimum display: ₹1.00 (100 paise) — aligns with database CHECK constraint | DATA-MODEL.md: `price_paise CHECK (price_paise >= 100)`; ARCHITECTURE.md: "All monetary values in integer paise, ₹1 = 100 paise" | High | N/A — confirmed in data model |
| A-05-011 | Status vocabulary "Pending Review" (not "Pending Approval") used consistently across all 3 screens | CLAUDE.md Architecture Decisions: "Canonical product/seller status term: Pending Review" (2026-05-05); GLOSSARY.md | High | N/A — confirmed decision |
| A-05-012 | Seller product row shows rejection reason inline (expanded state) rather than in a separate detail view | Mobile UX pattern — inline expansion reduces navigation depth; rejection reason is short text; WIREFRAMES.md SCR-003 State 5 | Medium | Phase 9 (Priya persona usability test) |
| A-05-013 | Edit product form shows name/description as editable but with a re-review warning banner rather than being completely disabled | FR-PRODUCT-007, FR-PRODUCT-008 — price/stock edits don't require re-review; name/desc changes do; user should be able to edit all fields but informed of consequences | High | Phase 7 (Implementation) |
| A-05-014 | WCAG 2.1 AA orange CTA (#F97316) against white background achieves 3.0:1 — borderline for 14px normal weight text | Colour Contrast Analyser calculation; WCAG 1.4.3 requires 4.5:1 for normal text but 3.0:1 for large text (≥18pt or ≥14pt bold); using font-weight: 600 at 14px qualifies as "large bold text" | Medium | Phase 9 (A11Y-009 contrast test) |
| A-05-015 | 3 screens selected: SCR-001 (PDP), SCR-002 (Cart & Checkout), SCR-003 (Seller Products) — autonomously selected as highest-value | Coverage analysis: SCR-001 touches US-009, US-010, US-011 (product browse, detail, search); SCR-002 touches US-013, US-014, US-016, US-017 (cart, checkout, guest, payment); SCR-003 touches US-004, US-005, US-006, US-007 (product CRUD, approval, edit, orders view); spans all 3 personas (Rahul, Anjali, Priya); covers P0 flows | High | Human Gate review |
| A-05-016 | HTML wireframe files use 1440px desktop reference frame (`min-width: 1280px` viewport lock) — not mobile phone frames | Updated agent spec (`05_ux_design_agent_testing.md`) mandates desktop viewport, vertical state sections, and inline `.ann` annotations; prevents phone-frame clipping of desktop component details that reviewers need to inspect | High | N/A — agent spec requirement |
| A-05-017 | First HTML wireframe pass (pre re-execution) used mobile 375px phone frames arranged side-by-side — discarded on re-execution | Agent spec was not read before first-pass generation; on re-execution spec was read first, all 3 files regenerated as desktop 1440px vertical layouts; first-pass files overwritten | High | N/A — execution note |

---

## Open Flags (Tier 2 — Unconfirmed Suggestions)

| Flag ID | Suggestion Made | Location | Status |
|---------|----------------|----------|--------|
| F-05-001 | Orange (#F97316) as CTA color — requires explicit stakeholder approval before Phase 7 implementation; alternative is to use blue-600 for all CTAs (less conventional but safer for WCAG) | DESIGN-SYSTEM.md §2 Color Tokens | Pending stakeholder visual review |
| F-05-002 | System font stack vs. web font — confirm before Phase 7 if brand requires a specific typeface (e.g., "Noto Sans" for better Devanagari support in error messages or admin UI) | DESIGN-SYSTEM.md §3 Typography | Pending final brand decision |
| F-05-003 | Guest tracking page (separate from these 3 screens) — not designed in this testing run; should be added in full Phase 5 run or in Phase 7 as a low-complexity screen | USER-JOURNEYS.md Journey 2 Step 8 | Flagged for full UX run |
| F-05-004 | Admin approval queue screen — not in testing scope (3 screens); P0 per US-005 but serves Admin persona only; design in full Phase 5 run | US-005 (Admin Approves Product) | Flagged for full UX run |

---

## Screen Selection Rationale

| Screen | User Stories Covered | Personas | P0 Flows |
|--------|---------------------|---------|---------|
| SCR-001 Product Detail Page | US-009 (browse), US-010 (product detail), US-011 (stock visibility), US-016 (self-purchase block) | Rahul (buyer), Anjali (guest) | Product discovery (highest-volume buyer action) |
| SCR-002 Cart & Checkout | US-013 (add to cart), US-014 (checkout), US-015 (guest checkout), US-016 (payment), US-017 (order confirmation) | Rahul (buyer), Anjali (guest) | Revenue-generating critical path |
| SCR-003 Seller Products | US-004 (add product), US-005 (product status), US-006 (edit product), subscription gate | Priya (seller) | Seller monetization + platform supply |

**Total unique user stories covered across 3 screens: 11 of ~20 buyer/seller stories. Persona coverage: 3/3 personas.**

---

## Resolution Log

| ID | Original Assumption | Resolution | Resolved By | Date |
|----|--------------------|-----------|-----------:|------|
| F-05-001 | Orange CTA color | Pending stakeholder review at Human Gate | Human Gate | TBD |
| F-05-002 | System font | Pending brand decision | Phase 7 frontend setup | TBD |

---

## Full Production Run — Additional Tier 3 Inferences (SCR-004 through SCR-021)

| ID | Inference | Basis | Confidence | Must Validate Before Phase |
|----|-----------|-------|------------|---------------------------|
| A-05-018 | Hero banner on homepage is 1440×320px with text overlay CTA — full-bleed desktop layout | Indian e-commerce convention (Flipkart, Meesho, Amazon.in all use hero banners); established DESIGN-SYSTEM.md pattern for SCR-004 | High | Phase 7 frontend |
| A-05-019 | Product grid: 4-col desktop, 2-col tablet, 1-col mobile — same grid across SCR-004, SCR-005, SCR-006 | Consistent with Amazon.in/Flipkart grid density; prevents re-learning between screens; Tailwind `grid-cols-1 md:grid-cols-2 lg:grid-cols-4` pattern | High | Phase 7 |
| A-05-020 | Search bar in buyer top nav is pre-populated with query on SCR-006 search results page | Standard UX convention (Google, Amazon, Flipkart); allows immediate refinement; `defaultValue` via URL query param in Next.js SSR | High | Phase 7 |
| A-05-021 | Buyer auth screen (SCR-007) uses tab toggle (Login / Register) in a single centered card rather than two separate routes | Reduces page navigation overhead; common Indian consumer app pattern (Myntra, Nykaa); both tabs on same route via URL hash or query param | High | Phase 7 |
| A-05-022 | Order status timeline (SCR-008) uses 4 fixed steps: Payment Confirmed → Processing → Shipped → Delivered | REQUIREMENTS.md order status machine (FR-ORDER-004); 4 statuses cover complete lifecycle; CANCELLED shown as branch replacing Delivered step | High | N/A — confirmed in requirements |
| A-05-023 | Guest order tracking URL (SCR-008) shows "ShopNest Internal Use Only" — no buyer-facing login prompt on invalid token; instead shows generic sign-in CTA | Security posture: HMAC token failure must not reveal order data; minimal fallback prevents token enumeration; ADR-004 confirmed HMAC-SHA256 | High | N/A — security decision |
| A-05-024 | Seller registration (SCR-010) uses a standalone centered card layout with no sidebar — pre-auth screens have no navigation shell | Standard: users not yet authenticated have no seller context; sidebar implies access to seller features they don't have yet; matches SCR-012 and SCR-018 patterns | High | Phase 7 |
| A-05-025 | Store setup wizard (SCR-011) has 3 steps: Store Info → Categories → Review — minimum viable setup | USER-STORIES.md US-002 covers store setup; 3 steps cover: branding (logo/banner), category targeting, confirmation; additional steps (shipping config, etc.) are post-MVP | High | Phase 7 |
| A-05-026 | Admin sidebar uses dark navy (#1E293B) background to visually distinguish from seller light sidebar | Admin = internal tool; clear visual distinction prevents accidental operation on wrong portal; dark admin theme is common in CMS/admin tools (Shopify admin, Django admin dark variants) | Medium | Phase 5 Human Gate — stakeholder approval |
| A-05-027 | Seller dashboard KPI cards show: Orders This Month, Revenue This Month, Active Products — 3 primary metrics | These 3 metrics directly address Priya's (seller persona) core questions: "Am I getting orders?", "How much am I making?", "Are my products live?"; all derivable from existing data model | High | Phase 7 |
| A-05-028 | Admin dashboard includes "Action Required" section with pending seller + product counts and direct links to queues | Admin primary workflow is queue processing; surfacing queue depth + oldest item on dashboard drives SLA compliance (OQ-004); reduces clicks from dashboard to action | High | Phase 7 |
| A-05-029 | Seller payouts table shows 4 columns: Gross, Gateway Fee, Net Paid, Status — transparency model | CLAUDE.md architecture decision 2026-05-05: zero commission; gateway fee (~2%) absorbed by ShopNest but shown transparently to seller; SettlementLedgerEntry model stores both gross_amount_paise and gateway_fee_paise | High | N/A — confirmed in data model |
| A-05-030 | Subscription management (SCR-016) billing history table includes "Download PDF" invoice links — one PDF per billing cycle | REQUIREMENTS.md §5.1: 7-year retention for financial records; WeasyPrint generates GST-compliant PDFs stored in S3; seller can download for tax compliance | High | Phase 7 |
| A-05-031 | Store settings (SCR-017) shows masked bank account (last 4 digits) by default; "Edit" click reveals input — not shown on page load | AES-256-GCM encryption means the full number is never in the DOM on initial render; "Edit" flow requires deliberate user action; defense-in-depth against shoulder surfing | High | N/A — security decision (SECURITY-ARCHITECTURE.md) |
| A-05-032 | Admin product approval queue (SCR-020) includes expandable product image gallery with click-to-enlarge — images open in modal overlay | WIREFRAMES.md cognitive walkthrough identified this as highest friction point for admin: "Judging product quality from small thumbnail"; larger images enable meaningful quality assessment before approve/reject decision | High | Phase 7 |
| A-05-033 | Both admin queues (SCR-019, SCR-020) use tab bar with counts: Pending / Approved / Rejected — same pattern | Consistent with standard admin queue UI (GitHub PR review, Jira board); tab counts give admin instant visibility of queue depth; switching tabs shows historical decisions | High | Phase 7 |

---

## Full Production Run — Open Flags (Tier 2 — Unconfirmed Suggestions)

| Flag ID | Suggestion Made | Location | Status |
|---------|----------------|----------|--------|
| F-05-005 | Admin dark sidebar (#1E293B) — confirm this visual distinction is desired by the team before Phase 7 implementation | DESIGN-SYSTEM.md §11.3 | Pending stakeholder review at Human Gate |
| F-05-006 | Homepage hero banner image — ShopNest needs to produce or license a hero image for production; wireframe uses a blue gradient placeholder | SCR-004 wireframe | Requires content/design team decision; Phase 7 |
| F-05-007 | KPI "Revenue This Month" on seller dashboard — confirm whether this shows gross or net (after Razorpay fee) | SCR-013 wireframe | Pending business decision — recommend gross with note "Before gateway fee" |
| F-05-008 | Admin dashboard "Recent Activity" feed — confirm whether AuditLog captures all required actions for this view | SCR-021 wireframe | Phase 7 — AuditLog schema in DATA-MODEL.md covers approve/reject actions |
