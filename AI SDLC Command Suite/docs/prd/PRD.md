# Product Requirements Document (PRD) — ShopNest
**Version:** 1.0 — Draft
**Status:** Draft
**Product Owner:** Product Owner (role — assign before Phase 4)
**Tech Lead:** Full-Stack Lead (role — assign before Phase 4)
**Last Updated:** 2026-05-04
**Living Document:** Yes — changes require version increment and owner sign-off

---

## Document Control

| Version | Date | Author | Change Summary | Approved By |
|---------|------|--------|----------------|-------------|
| 1.0 | 2026-05-04 | PRD Agent (Phase 3) | Initial synthesis from Phase 1 + Phase 2 artifacts | Pending |

---

## Conflict & Gap Resolution Log

| ID | Type | Source | Resolution | Impact |
|----|------|--------|------------|--------|
| C-001 | Conflict | Phase 1 "100% revenue" vs Phase 2 BR-005 "net of Razorpay fee" | Zero *platform* commission. Razorpay gateway fee (~2%+GST) is standard; sellers receive order amount net of gateway fee. | Clarified in §6, BR-005 |
| C-002 | Conflict | Seller status terminology: "Pending Review" vs "Pending Admin Approval" | Canonical status = "Pending Review" for both seller accounts and product listings | Applied throughout PRD |
| G-001 | Gap | Rejected seller re-registration path | Rejected seller email blocked from re-registration; must contact support to appeal | Added to §13 Open Questions |
| G-002 | Gap | EMI payment method: Phase 1 VP vs Phase 2 FR-CHECKOUT-004 | EMI included as available Razorpay checkout option (bank-dependent buyer eligibility; no ShopNest gating required) | Added to §8 Feature 4 |

---

## 1. Executive Summary

ShopNest is a hosted, multi-vendor SaaS marketplace for Indian SME retailers. It enables fashion, lifestyle, and home-goods sellers to launch a branded, mobile-optimised online storefront and accept India-native payments (UPI, cards, NetBanking, EMI via Razorpay) within 30 minutes of signing up — for a flat monthly subscription with zero per-transaction commission.

Buyers discover products across all ShopNest sellers on a single marketplace domain, browse server-side-rendered product and category pages (SEO-optimised), and complete checkout as a guest or registered buyer. Sellers manage their store, products, orders, and view basic analytics from a seller dashboard. A Platform Admin approves seller accounts and product listings before they go live.

ShopNest targets the structural gap between India's large-marketplace duopoly (Amazon/Flipkart, 15–40% commission, brand-opaque) and self-hosted solutions (WooCommerce, technically inaccessible to SMEs). Its revenue model is a flat monthly subscription per seller — decoupled from GMV, making it the seller's financial ally rather than competitor.

### 1.1 Background & Context

**Market driver:** India has ~63 million MSMEs. An estimated 850,000 SME sellers with 50–500 SKUs are actively seeking or using digital sales channels. The dominant solutions — Amazon/Flipkart and WhatsApp — serve opposite ends of the spectrum: massive reach with high commissions, or zero infrastructure with no scalability. No India-native, zero-commission SaaS marketplace serves the middle.

**Competitive landscape:** Amazon India and Flipkart command marketplace discovery but extract 15–40% commission. Shopify offers branded storefronts but is USD-priced and UPI-unfriendly. WooCommerce requires technical self-hosting. WhatsApp+spreadsheet is the default fallback — free but unable to handle >50 orders/month. Full analysis in `docs/ideation/COMPETITIVE-ANALYSIS.md`.

**Success looks like:** 100 active paying seller subscriptions within 6 months of public launch, monthly churn below 5%, and at least one cohort of sellers receiving weekly payouts — proving the end-to-end transaction loop works.

**Why now (market timing):**
1. **UPI ubiquity:** UPI crossed 10 billion monthly transactions in India (NPCI 2024). Buyers in Tier 1–2 cities complete purchases via UPI QR without a second thought — the payment friction that killed early Indian e-commerce is gone.
2. **Social commerce maturity:** Instagram and WhatsApp selling grew 3× from 2020–2023 among Indian SMEs (post-COVID digital shift). Sellers have proven buyer demand via social channels but have no structured checkout. The gap between "I can sell" and "I can scale" is exactly what ShopNest fills.
3. **Competitive squeeze:** Amazon India raised seller fees in 2023–2024; Shopify increased USD pricing in 2023. Both moves pushed marginal sellers to look for alternatives.
4. **Regulatory legitimization:** Consumer Protection (E-Commerce) Rules 2020 formalize the "marketplace e-commerce entity" category — giving legitimacy to alternatives and raising barriers against pure WhatsApp commerce.
5. **Team positioning:** The declared stack (Django + Razorpay + Next.js + AWS) maps directly to the India e-commerce infrastructure playbook. No new technology risk; execution risk only.

---

## 2. Problem Statement

> **Indian SME sellers (50–500 SKUs) who want to establish a professional online sales channel struggle to find a platform that is simultaneously affordable, India-payment-native, and technically accessible — because dominant marketplaces (Amazon/Flipkart) extract 15–40% commission and commoditise their brand, while self-hosted solutions (WooCommerce) exceed their technical capability and informal channels (WhatsApp + spreadsheets) cannot scale — which results in sellers either surrendering margin to marketplaces, remaining stuck at informal-channel scale, or abandoning e-commerce entirely.**

*Source: `docs/ideation/PROJECT-CONCEPT.md`*

### 2.1 Product Vision

```
For Indian SME retailers (50–500 product SKUs) in Tier 1–2 cities
Who need to sell online without surrendering margins to large marketplaces or
  hiring a developer,
ShopNest is a multi-vendor SaaS marketplace platform
That enables sellers to launch a branded, mobile-optimised storefront and accept
  UPI/card payments in under 30 minutes — with full order and inventory management
  included.
Unlike Amazon India and Flipkart (which charge 15–40% commission and own the
  customer relationship) and WooCommerce (which requires self-hosting and technical
  expertise),
Our product gives sellers a zero-commission, flat-subscription model with complete
  ownership of their storefront, customer data, and brand identity.
```

### 2.2 Proposed Solution

ShopNest is a hosted multi-vendor SaaS marketplace. Sellers pay a flat monthly subscription (single plan, all features included), complete a guided store setup wizard, and immediately access product listing, order management, and payout settlement. Buyers discover products across all sellers on ShopNest's shared marketplace domain, browse SSR-rendered pages, and check out via Razorpay — with or without creating an account.

The approach was chosen over two alternatives:
- **White-label store builder (Approach B):** Rejected — no cross-seller buyer aggregation, no network effects, operationally heavier for a 3-person team.
- **WhatsApp-integrated order hub (Approach C):** Rejected — structural dependency on Meta's API policies; no SEO benefit; guest checkout not cleanly implementable.

The multi-vendor marketplace creates compounding network effects: more sellers attract more buyers, which attracts more sellers. This is the structural moat that neither individual store builders nor WhatsApp can replicate.

### 2.3 Current Challenges

| Persona | Current Workaround | Why It Breaks Down |
|---------|-------------------|-------------------|
| SME Seller | Amazon/Flipkart seller account | 15–40% commission; customer relationship owned by marketplace; brand invisible |
| SME Seller | WhatsApp Business + spreadsheet | No discoverability; no guest checkout; breaks at >50 orders/month |
| SME Seller | WooCommerce on shared hosting | Requires developer; no managed hosting; ongoing maintenance burden |
| Indian Buyer | Amazon/Flipkart + social media | No unified way to purchase from indie/boutique sellers with full buyer protection |

### 2.4 Strategic Alignment

ShopNest's goals map to the following informal objectives (team does not use a formal OKR framework):

| Objective | Key Result | ShopNest's Contribution |
|-----------|-----------|------------------------|
| Establish ShopNest as a sustainable SaaS business | 100 active paying sellers within 6 months post-launch | The entire seller onboarding + subscription flow |
| Prove the transaction loop works | ≥1 order placed and fulfilled within 2 weeks of launch | Guest checkout + Razorpay + order fulfillment flow |
| Build a marketplace buyers return to | Buyer repeat purchase rate > 25% within 30 days | SSR product pages (SEO), smooth checkout UX, order tracking |
| Keep infrastructure cost viable | AWS cost per seller < ₹2,000/month at 100 sellers | Managed services (Fargate, RDS, ElastiCache, S3) architecture |

### 2.5 Four Product Risks Summary

| Risk | Level | Key Mitigation |
|------|-------|---------------|
| Value | Medium | Beta seller program pre-launch; 30-day free trial; "first order received" as activation trigger |
| Usability | Medium-High | Guided onboarding wizard; usability test with 5 sellers before launch; WCAG 2.1 AA compliance |
| Feasibility | Medium | Razorpay Payouts API spike in Sprint 1; strict scope gate; no post-MVP features; managed-service stack |
| Business Viability | Medium | Pricing validation interviews before launch; Razorpay fee absorption modeled; CAC measured from first 20 sellers |

**Full risk assessment: §2.5 detail below.**

#### Value Risk — Medium
*Risk:* SME sellers may not see ₹1,999/month as justified vs. free WhatsApp. The switching cost from "free + familiar" to "paid + structured" requires a compelling first-order experience.
*Evidence of mitigation:* Phase 1 competitive analysis confirms no India-native zero-commission SaaS option exists. The beachhead segment (Instagram/WhatsApp sellers with >30 orders/month) is already at the breaking point of their current workaround.
*Remaining plan:* Run 10–15 structured beta seller pricing interviews in Q3 2026 before public launch. Set activation metric: "seller receives first order within 30 days of going live" as the core value proof point. Offer 30-day free trial to reduce signup friction. *(Assumption A-01-016)*

#### Usability Risk — Medium-High
*Risk:* Target sellers are SME owners, not technical users. 30-minute onboarding is the promise; a confusing wizard or unclear product approval process could cause drop-off before first listing.
*Evidence of mitigation:* Guided wizard (FR-SELLER-001) forces linear onboarding. Admin product approval (FR-PRODUCT-003) provides a human safety net. Mobile-first design (NFR-USA-001) matches the device the seller will use.
*Remaining plan:* Moderated usability test with 5 representative sellers (from beachhead segment) before beta launch. Measure: seller can complete registration → store setup → first product submission in ≤30 minutes unassisted. Fix any step where >2/5 testers fail or pause. *(NFR-USA-004)*

#### Feasibility Risk — Medium
*Risk:* Multi-vendor marketplace + guest checkout + Razorpay subscription billing + Razorpay Payouts API in 6.5 months for a 3-person team is ambitious.
*Evidence of mitigation:* Phase 1 feasibility rated Technical as Green. Stack is proven (Django, Next.js, Razorpay, AWS). Managed services (Fargate, RDS, ElastiCache) reduce DevOps overhead. Single flat subscription plan (confirmed Phase 2) eliminates feature-gating complexity.
*Remaining plan:* Spike Razorpay Payouts API integration in Sprint 1 of Phase 7. Razorpay Payouts requires a separate activation on ShopNest's account — confirm before implementation begins (Open Question OQ-002). Strict scope gate: no feature begins unless in the approved In-Scope list.

#### Business Viability Risk — Medium
*Risk:* Subscription pricing unvalidated; seller CAC unknown; Razorpay gateway fee (~2%+GST) absorbed from subscription margin at thin margins at 100 sellers (gross margin ~17%).
*Evidence of mitigation:* Break-even at 83 sellers (AWS cost = ₹1,66,000; MRR at 83×₹1,999 = ~₹1,66,000). Margin improves to ~60% at 500 sellers. SOM model is conservative (0.05% of SAM in Year 1).
*Remaining plan:* Pricing validation before public launch (OQ-001). Measure CAC from first 20 sellers. If month-3 churn >8%, pause acquisition and fix retention before scaling. AWS Cost Explorer alert at 80% of monthly budget.

---

## 3. Product Goals & Success Metrics

### 3.1 Desired Outcome

> **Achieve 100 active paying seller subscriptions within 6 months of public launch, with monthly churn below 5% — proving ShopNest delivers sufficient ongoing value to retain sellers beyond the first invoice.**

### 3.2 Business Goals

| Goal | Metric | Baseline | Target | Timeline |
|------|--------|----------|--------|----------|
| Sustainable MRR | Monthly Recurring Revenue | ₹0 | ₹1,99,900 (100 × ₹1,999) | 6 months post-launch |
| Seller retention | Monthly churn rate | — | < 5% | From Month 3 |
| Seller lifetime value | 12-month LTV | — | > ₹23,988 | Month 12 |
| Infrastructure efficiency | AWS cost per seller | — | < ₹2,000/seller/month at 100 sellers | Month 6 |
| Gross margin path | Gross margin % | ~17% at launch (100 sellers) | > 50% at 500 sellers | Month 18 |

### 3.3 User Goals

| Goal | Metric | Baseline | Target | Timeline |
|------|--------|----------|--------|----------|
| Seller activation | Store setup completion rate | 0% | > 80% | Month 1 |
| Time to first value (seller) | Time from signup to first product listed | — | < 30 minutes | Month 1 |
| Seller activation depth | Sellers receiving ≥1 order within 30 days | 0% | > 70% | Month 2 |
| Order fulfillment | Orders marked Shipped within 48 hours | — | > 95% | Month 3 |
| Seller satisfaction | Seller NPS | — | > 40 | Month 3 |
| Buyer conversion | Cart-to-order conversion rate | — | > 3% | Month 3 |
| Guest checkout | Guest checkout completion rate | — | > 60% | Month 3 |
| Buyer retention | Repeat purchase within 30 days | — | > 25% | Month 6 |
| Buyer trust | Order tracking engagement | — | > 50% of buyers view tracking at least once | Month 3 |

### 3.4 Technical Goals

| Goal | Metric | Target | Measurement |
|------|--------|--------|-------------|
| API performance | P95 response time | < 200ms | CloudWatch load test |
| Page performance | Product page LCP (mobile 4G) | < 2.5s | Core Web Vitals |
| Reliability | Monthly uptime | ≥ 99.9% | CloudWatch alarm |
| Payment reliability | Checkout error rate | < 1% | Razorpay + app logs |
| Payment success | Razorpay payment success rate | ≥ 97% | Razorpay dashboard |
| Scale readiness | Concurrent users (no degradation) | ≥ 1,000 | k6/Locust load test |
| Code quality | Test coverage | ≥ 80% backend + frontend | CI pipeline reports |

*Full NFR specification: `docs/requirements/REQUIREMENTS.md §4`*

### 3.5 Definition of Success

**At launch (Day 1):**
- All Must Have user stories passing acceptance tests
- End-to-end transaction loop verified: seller lists → buyer purchases → seller ships → buyer tracks
- Razorpay checkout, subscription billing, and payout APIs tested in production
- CloudWatch dashboards and alerts configured
- GST subscription invoice generation functional and legally compliant

**At 30 days (Leading indicators):**
- ≥ 10 active sellers (products listed, ≥1 order received)
- ≥ 1 weekly payout cycle executed successfully
- Seller support ticket volume < 2 tickets/seller/month
- No P0 (data loss / payment failure) incidents

**At 90 days (Full KPI validation):**
- ≥ 20–30 paying sellers; churn rate measured and < 5%
- Cart-to-order conversion rate measurable (minimum 100 checkout sessions)
- Pricing validation complete: confirm or adjust ₹1,999/month

**At 12 months:**
- 500 sellers target (base case SOM)
- MRR ≥ ₹9,99,500 (500 × ₹1,999)
- Gross margin trending toward 50%
- Decision point: invest in Phase 2 features (logistics integration, analytics, reviews)

---

## 4. Scope

### 4.1 In Scope (MVP — by 2026-10-31)

**Seller Side:**
- Self-registration with store setup wizard (name, logo, banner, categories)
- Monthly flat-rate subscription via Razorpay Subscriptions
- Product management: add/edit/delete; image upload to S3/CloudFront; price, stock, category
- Admin product approval workflow (submit → pending review → approved/rejected)
- Order management: view orders, confirm, ship (AWB entry), mark delivered
- Settlement ledger and weekly payout via Razorpay Payouts API (every Monday 09:00 IST)
- Basic analytics: revenue, order count, AOV (last 7/30 days); top 5 products
- Low-stock alerts (≤5 units)
- GST-compliant subscription invoice generation and download
- Payout history (last 12 months)

**Buyer Side:**
- Homepage, category pages, product detail pages (SSR — SEO-optimised)
- Keyword search (PostgreSQL full-text search)
- Cart (guest and registered, Redis-persisted, 24h TTL)
- Guest checkout (name, email, phone, address — no account required)
- Registered buyer checkout with saved address
- Razorpay checkout: UPI, cards (credit/debit), NetBanking, EMI (bank-dependent)
- Order cancellation before seller confirmation (with full Razorpay refund)
- Order tracking page (publicly accessible via HMAC URL in confirmation email)
- Transactional email notifications (confirmation, shipped, cancelled)
- Buyer account registration and order history

**Platform Admin:**
- Seller approval queue (approve/reject with reason)
- Product approval queue (approve/reject/remove with reason)
- Seller account management (suspend/reactivate)
- Platform overview dashboard (sellers, products, orders, GMV)

### 4.2 Out of Scope (This Release)

| Feature | Rationale | Target |
|---------|-----------|--------|
| Product reviews and ratings | Complex moderation; not needed for initial seller trust | Post-MVP |
| Recommendation engine / personalisation | Requires ML + sufficient data volume | Post-MVP |
| Logistics carrier integration (Shiprocket, Delhivery) | Sellers manage own shipping; AWB manual entry in MVP | Post-MVP |
| Mobile native app (iOS/Android) | Mobile-responsive web is MVP; native app is separate investment | Post-MVP |
| Multi-language support (Hindi, Tamil, etc.) | English-first MVP; regional expansion is post-MVP strategic decision | Post-MVP |
| Multi-currency / international shipping | India-only scope | Post-MVP |
| Wishlists / saved items | Convenience; not on critical buyer path | Post-MVP |
| Loyalty points / referral / affiliate programs | Nice-to-have; not core buyer journey | Post-MVP |
| Advanced seller analytics / BI | Requires BI tooling beyond MVP scope | Post-MVP |
| Seller-to-seller or seller-to-buyer messaging | Not required for core transaction loop | Post-MVP |
| Seller review rejection appeal workflow | Manual contact-support flow for MVP | Post-MVP |
| Guest order history linking to account | Requires post-registration backfill | Post-MVP |

---

## 5. Market Context

### 5.1 Target Beachhead Segment

**Who:** Independent fashion, lifestyle, and home goods sellers in Indian Tier 1–2 cities (Mumbai, Bangalore, Delhi, Hyderabad, Pune, Jaipur) currently selling via Instagram DMs and WhatsApp to 200–2,000 followers.

**Why most acutely underserved:**
- Already have proven buyer demand (social media followers) — no awareness problem
- Current workaround (Instagram DM → WhatsApp UPI → manual fulfillment) breaks at >30 orders/month
- Amazon/Flipkart commission eliminates handmade/boutique item margin
- WooCommerce is too technical; Shopify is USD-priced

**Why this segment creates momentum:**
- Fashion/lifestyle sellers are social-media-active — their ShopNest storefront link shared with Instagram/WhatsApp audience drives organic buyer traffic
- Seller success stories in this category are visually shareable — ideal for content marketing
- Adjacent categories naturally follow (electronics accessories, artisanal food, personalised gifts)

### 5.2 Market Size

| Market | Definition | Size | Source |
|--------|-----------|------|--------|
| TAM | All Indian product-selling, internet-capable MSMEs | ~15.1M sellers / ₹3.62L Cr/year | Ministry of MSME 2023 + derived |
| SAM | SMEs with 50–500 SKUs, Tier 1–2 cities, seeking digital sales | ~850K sellers / ₹2,039 Cr/year | Phase 1 bottom-up calculation |
| SOM (Year 1) | Referral-led acquisition, 3-person team, India-only | 500 sellers / ₹59.97L/year | Phase 1 base case |
| SOM (Year 2) | Scaled referral + content marketing | 2,000 sellers / ₹2.4 Cr/year | Phase 1 base case |

*Full sizing with assumptions: `docs/ideation/MARKET-SIZING.md`*

### 5.3 Competitive Position

**Gap statement:** "All existing solutions fall short at giving Indian SME sellers a zero-commission, branded storefront with India-native UPI checkout that requires no technical setup — because Amazon/Flipkart are commission-dependent, Shopify is western-market-priced, and WooCommerce requires developer expertise. ShopNest fills the 'Shopify for India, minus the commission' slot that remains genuinely open in 2026."

*Full competitive matrix: `docs/ideation/COMPETITIVE-ANALYSIS.md`*

---

## 6. Target User Personas

### Persona 1: Priya — SME Seller (Store Owner)

| Attribute | Detail |
|-----------|--------|
| **Background** | 29-year-old boutique clothing seller, Bangalore. Runs a women's fashion label with 120 SKUs. Sells via Instagram DMs and WhatsApp. Smartphone-primary; uses Google Sheets for inventory. Low technical proficiency — cannot configure WordPress. |
| **Primary Job (Functional)** | List products online, receive orders, get paid, manage inventory |
| **Frustrations** | Amazon's 25% commission eats her margin on ₹600 kurtis. WhatsApp orders get lost. Customers ask "do you have stock?" — she checks manually. No order tracking to share with buyers. |
| **Switching Moment** | She gets >30 orders/week via WhatsApp and misses 4 orders in one week due to chaos. She searches "sell online without Amazon commission India." |
| **Success Scenario** | Sets up her ShopNest store in 25 minutes on Saturday morning. Lists 10 products by Sunday. Receives first order on Monday. Sees ₹3,200 hit her bank account on the following Monday payout. |
| **Environment** | Android smartphone (Samsung Galaxy A), 4G/WiFi; uses Instagram daily; comfortable with Razorpay-style payment flows |

### Persona 2: Anjali — Guest Buyer

| Attribute | Detail |
|-----------|--------|
| **Background** | 26-year-old working professional, Mumbai. Shops online 2–3× per week. Discovers products via Instagram and WhatsApp groups. Has an Amazon account but prefers buying from independent creators. |
| **Primary Job (Functional)** | Find and buy unique/indie products quickly, without creating yet another account |
| **Frustrations** | Instagram sellers say "DM me to order" — manual, slow, no order tracking. She's been burned by WhatsApp sellers who didn't deliver. |
| **Switching Moment** | A seller shares her ShopNest link in a WhatsApp group. Anjali sees a real product page with a "Buy Now" button and UPI checkout. |
| **Success Scenario** | Discovers product, adds to cart, pays via UPI in 3 minutes without creating an account. Receives confirmation email. Tracks shipment via the link in the email. |
| **Environment** | iPhone (Safari), 4G; pays via PhonePe/Google Pay UPI; expects checkout to feel like Zomato or Swiggy |

### Persona 3: Rahul — Registered Buyer

| Attribute | Detail |
|-----------|--------|
| **Background** | 28-year-old tech professional, Pune. Shops regularly for home goods and electronics accessories. Has accounts on Amazon, Flipkart, and several D2C brands. Values convenience and order history. |
| **Primary Job (Functional)** | Purchase from indie sellers with a checkout experience as smooth as Amazon, with a saved address |
| **Frustrations** | Small sellers don't have structured checkout. Every seller's website looks different and inspires varying confidence. |
| **Switching Moment** | Finds a home decor seller on ShopNest; creates an account because he wants order history for future reference. |
| **Success Scenario** | Saves his address on first purchase. Returns for a second purchase — address is pre-filled, completes in 90 seconds. |

### Persona 4: Admin — Platform Admin

| Attribute | Detail |
|-----------|--------|
| **Background** | ShopNest team member (likely the full-stack lead in MVP phase). Responsible for seller onboarding quality, product moderation, and platform health. |
| **Primary Job (Functional)** | Keep the marketplace free of bad actors and low-quality listings; onboard good sellers quickly |
| **Frustrations** | Manual approval queues that grow faster than the team can process. |
| **Success Scenario** | Product approval queue is processed within 24 hours. No prohibited items ever go live. Seller suspension is instant when needed. |

---

## 7. Non-Functional Requirements Summary

| ISO 25010 Category | Key Requirement | Target | Full Spec |
|-------------------|----------------|--------|-----------|
| Performance Efficiency | API P95 response time | < 200ms | NFR-PE-001 |
| Performance Efficiency | Product page LCP (mobile 4G) | < 2.5s | NFR-PE-002 |
| Performance Efficiency | Search P95 (≤50K products) | < 500ms | NFR-PE-005 |
| Performance Efficiency | Concurrent users | ≥ 1,000 | NFR-PE-006 |
| Reliability | Monthly uptime | ≥ 99.9% | NFR-REL-001 |
| Reliability | RTO | < 1 hour | NFR-REL-002 |
| Reliability | Webhook idempotency | Zero duplicate orders | NFR-REL-005 |
| Security | Data in transit | TLS 1.2+ (TLS 1.3 preferred) | NFR-SEC-001 |
| Security | Authentication | RS256 JWT, access 24h / refresh 30d | NFR-SEC-003 |
| Security | Card data | Zero raw card data stored | NFR-SEC-007 |
| Security | Multi-tenant isolation | Zero cross-seller data leaks | NFR-SEC-005 |
| Usability | Mobile responsiveness | ≥ 375px fully functional | NFR-USA-001 |
| Usability | Accessibility | WCAG 2.1 Level AA | NFR-USA-003 |
| Usability | Seller onboarding time | < 30 minutes to first product | NFR-USA-004 |
| Maintainability | Test coverage | ≥ 80% backend + frontend | NFR-MAINT-001/002 |
| Maintainability | API documentation | OpenAPI 3.0 spec, 100% coverage | NFR-MAINT-003 |
| Compatibility | API standard | RESTful, /api/v1/, snake_case JSON | NFR-COMPAT-001 |
| Portability | Containerization | OCI-compliant Docker images | NFR-PORT-001 |

*Full specification: `docs/requirements/REQUIREMENTS.md §4`*

---

## 8. Feature Specifications

> Features are organized by user journey, not by engineering layer.

---

### Journey 1: Seller Onboarding & Subscription

#### Feature 1.1 — Seller Registration

**One-line description:** New seller creates an account with business details; enters the admin approval queue.

**User Job Addressed:** "I want to start selling online without needing a developer or a marketplace account." (OPP-001)

**User Stories:** US-001, US-029 (admin rejection)

**Acceptance Criteria:**
- [ ] Seller submits: email, password (≥8 chars, ≥1 digit), full name, business name, store name, 10-digit phone, city, GSTIN (optional), 6-digit pincode → account created with status "Pending Review"
- [ ] Duplicate email → HTTP 409; no account created
- [ ] Password stored as bcrypt hash (cost factor ≥ 12); never logged
- [ ] New application visible in Admin Seller Approval Queue within 5 seconds

**Priority:** Must Have
**Dependencies:** AWS SES (approval notification), Platform Admin (reviews queue)
**Applicable NFRs:** NFR-SEC-003, NFR-SEC-004, NFR-USA-004
**Applicable Business Rules:** BR-014
**Feature Success Metric:** ≥ 80% of sellers who start registration complete all form fields and submit (drop-off rate < 20%)
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope (this feature):** GSTIN validation, Aadhaar verification, social login (post-MVP)

---

#### Feature 1.2 — Store Setup Wizard

**One-line description:** Guided, step-by-step wizard for new sellers to configure their store identity before listing products.

**User Job Addressed:** "I want my store to look professional without hiring a designer." (OPP-001)

**User Stories:** US-002, US-031

**Acceptance Criteria:**
- [ ] Wizard collects: store name (≤100 chars), logo (PNG/JPG ≤2MB), banner (PNG/JPG ≤5MB), up to 5 product categories
- [ ] Images uploaded to S3 and served via CloudFront; URLs in store record
- [ ] Seller blocked from product listing interface until wizard is complete
- [ ] Unsupported file types or oversized files → HTTP 400 with specific message
- [ ] After wizard: seller is redirected to subscription payment step

**Priority:** Must Have
**Dependencies:** AWS S3, Razorpay Subscriptions (next step)
**Applicable NFRs:** NFR-PE-002, NFR-USA-001, NFR-USA-004
**Applicable Business Rules:** BR-015
**Feature Success Metric:** > 80% of sellers who start the wizard complete all steps and proceed to subscription (funnel: wizard_step_1_completed → wizard_completed)
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Custom domain (post-MVP), custom theme/color scheme (post-MVP)

---

#### Feature 1.3 — Seller Subscription (Razorpay Subscriptions)

**One-line description:** Single flat-plan monthly subscription via Razorpay; manages activation, grace period, and suspension lifecycle.

**User Job Addressed:** "I want to pay a predictable flat fee, not lose margin per sale." (Business model — BR-005)

**User Stories:** US-003, US-036

**Acceptance Criteria:**
- [ ] Razorpay Subscription created on successful first payment; seller_status + subscription_status = Active
- [ ] `subscription.charge.failed` webhook → status = Payment Failed; seller notified; 7-day grace period begins
- [ ] Day 8 without payment → status = Suspended; all products hidden from marketplace
- [ ] `subscription.charged` (success) webhook → status restored to Active; products re-published
- [ ] GST-compliant PDF invoice generated per successful billing event; downloadable from dashboard

**Priority:** Must Have
**Dependencies:** Razorpay Subscriptions API, AWS SES, FR-SUBSCR-001 through FR-SUBSCR-007
**Applicable NFRs:** NFR-REL-005 (webhook idempotency), NFR-MAINT-005 (audit log)
**Applicable Business Rules:** BR-001, BR-007, BR-010, BR-013
**Feature Success Metric:** Subscription payment success rate ≥ 97%; grace-period-to-suspension transition executes correctly on day 8 (automated test)
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Tiered plans, annual billing, coupon/discount codes (all post-MVP)

---

### Journey 2: Product Listing & Moderation

#### Feature 2.1 — Product Management (Seller)

**One-line description:** Seller creates, edits, and deletes product listings; manages price and stock in real time.

**User Job Addressed:** "I want to manage my catalogue without a developer." (OPP-003, OPP-004)

**User Stories:** US-004, US-006, US-010

**Acceptance Criteria:**
- [ ] Create: name (≤200 chars), description (≤2,000 chars), price (INR ≥₹1), stock (int ≥0), category, 1–5 images (JPEG/PNG ≤5MB each) → product status = Pending Review
- [ ] Price/stock edit → product remains Active (no re-review); name/description/image edit → re-triggers Pending Review (BR-017)
- [ ] Stock → 0 → product auto-hidden from marketplace
- [ ] Stock ≤ 5 → low-stock alert visible in dashboard within 60 seconds
- [ ] Delete blocked if any order in status other than Delivered/Cancelled (BR-009)

**Priority:** Must Have
**Dependencies:** AWS S3, CloudFront, Admin Product Approval (Feature 2.2)
**Applicable NFRs:** NFR-SEC-009 (MIME validation), NFR-PE-001
**Applicable Business Rules:** BR-001, BR-009, BR-015, BR-017
**Feature Success Metric:** > 80% of sellers who click "Add Product" successfully submit for review (completion funnel); product_submitted_for_review event rate
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Product variants (size/color), bulk CSV import (post-MVP)

---

#### Feature 2.2 — Admin Product & Seller Approval

**One-line description:** Platform Admin reviews and approves or rejects seller registrations and product listings before they go live.

**User Job Addressed:** Platform integrity — ensures only legitimate sellers and appropriate products appear on ShopNest.

**User Stories:** US-005, US-021, US-022, US-023, US-029, US-030

**Acceptance Criteria:**
- [ ] Seller approval queue: shows all Pending Review sellers; approve → Active + email; reject with reason → email with reason
- [ ] Product approval queue: shows all Pending Review products with thumbnails; approve → Active, product live; reject with reason → seller notified by email
- [ ] Admin can remove any Active product at any time with reason; seller notified
- [ ] Admin can suspend/reactivate any seller; suspension hides all products immediately; reactivation re-publishes
- [ ] Admin endpoints return HTTP 403 for non-Admin JWTs

**Priority:** Must Have
**Dependencies:** FR-AUTH-008, FR-AUTH-011 (Admin role enforcement)
**Applicable NFRs:** NFR-SEC-010 (RBAC), NFR-SEC-005 (data isolation)
**Applicable Business Rules:** BR-001, BR-014
**Feature Success Metric:** Admin review queue processed within 24 hours (operational SLA — not a system metric)
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Automated AI moderation, bulk approval (post-MVP)

---

### Journey 3: Buyer Discovery & Browse

#### Feature 3.1 — Marketplace Browse & Search

**One-line description:** SSR-rendered homepage, category pages, and product detail pages; keyword search across all active products.

**User Job Addressed:** "I want to discover products from indie sellers without knowing they exist beforehand." (OPP-002)

**User Stories:** US-011, US-012, US-013, US-014

**Acceptance Criteria:**
- [ ] Homepage, category pages, and product detail pages render server-side (HTML includes product data visible to Googlebot)
- [ ] Product detail page: og:title, og:description, og:image meta tags present
- [ ] Stable URL slugs: product URL does not change when name is edited
- [ ] Out-of-stock products hidden from category and search results
- [ ] Suspended seller's products hidden from all buyer-facing surfaces
- [ ] Search: keyword across product name + description; PostgreSQL FTS; results ≤ 500ms P95 at 50K products

**Priority:** Must Have
**Dependencies:** Next.js App Router (SSR), PostgreSQL FTS index, Redis (product catalog cache)
**Applicable NFRs:** NFR-PE-002 (LCP < 2.5s), NFR-PE-005 (search ≤500ms), NFR-USA-001 (375px), NFR-USA-003 (WCAG 2.1 AA)
**Applicable Business Rules:** BR-001 (triple-gate visibility), BR-004 (INR only)
**Feature Success Metric:** Product page LCP < 2.5s on mobile 4G (measured via Core Web Vitals); search result click-through rate > 15%
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Faceted filtering (category + price range), sort by (popularity, price) — post-MVP

---

### Journey 4: Cart & Checkout

#### Feature 4.1 — Shopping Cart

**One-line description:** Persistent cart for guest and registered buyers; multi-seller cart supported; real-time totals.

**User Job Addressed:** "I want to collect items before deciding to pay." (OPP-002)

**User Stories:** US-015, US-026

**Acceptance Criteria:**
- [ ] Any visitor (no login) can add to cart; cart cookie issued; Redis TTL 24h
- [ ] Cart supports products from multiple sellers simultaneously
- [ ] Quantity update (max = stock); remove item; subtotal updates in real time
- [ ] Stock validation at checkout initiation; sold-out items removed with error message before proceeding

**Priority:** Must Have
**Dependencies:** Redis (ElastiCache), FR-CART-001 through FR-CART-007
**Applicable NFRs:** NFR-PE-001, NFR-USA-001
**Applicable Business Rules:** BR-008 (stock decremented at payment, not cart)
**Feature Success Metric:** Cart abandonment rate after stock validation error < 30% (users who encounter stock error and still checkout with remaining items)
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Cart sharing, saved-for-later, cart merging on login (post-MVP note: guest cart is NOT merged with registered cart in MVP — BR-018)

---

#### Feature 4.2 — Guest & Registered Checkout + Razorpay Payment

**One-line description:** Complete checkout flow for guest and registered buyers; payment via Razorpay (UPI, card, NetBanking, EMI); order created on webhook confirmation.

**User Job Addressed:** "I want to pay with UPI in seconds without being forced to create an account." (OPP-002, OPP-006)

**User Stories:** US-016, US-017, US-025, US-026

**Acceptance Criteria:**
- [ ] Guest checkout collects: name, email, 10-digit phone, full shipping address — no account creation required
- [ ] Registered buyer checkout: saved address pre-populated; option to enter new address
- [ ] Razorpay Order created before payment UI presented; amount in INR paise
- [ ] Payment confirmed ONLY via Razorpay `payment.captured` webhook with HMAC-SHA256 validation — client-side callback alone does NOT create an order
- [ ] Multi-seller cart → split into seller-specific sub-orders within a single transaction
- [ ] Stock decremented atomically at order creation; concurrent oversell prevented
- [ ] Raw card data never logged or stored at application layer
- [ ] EMI available as payment option within Razorpay checkout (bank-dependent buyer eligibility; no ShopNest gating required)
- [ ] `payment.failed` webhook → stock restored; order marked Payment Failed

**Priority:** Must Have
**Dependencies:** Razorpay Orders API, Razorpay Webhooks, FR-CHECKOUT-001 through FR-CHECKOUT-012
**Applicable NFRs:** NFR-PE-007 (checkout < 3 min), NFR-SEC-006 (webhook HMAC), NFR-SEC-007 (no card data), NFR-REL-005 (idempotency)
**Applicable Business Rules:** BR-004 (INR), BR-008 (stock decrement), BR-012 (no self-purchase)
**Feature Success Metric:** Guest checkout completion rate > 60%; payment success rate > 97%; checkout error rate < 1%
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Saved payment methods, CoD (Cash on Delivery), BNPL (post-MVP)

---

### Journey 5: Order Management & Fulfillment

#### Feature 5.1 — Order Tracking (Buyer)

**One-line description:** Publicly accessible order tracking page via HMAC URL in confirmation email; shows status and AWB.

**User Job Addressed:** "I want to know where my order is without creating an account." (OPP-006)

**User Stories:** US-018, US-019, US-020, US-027

**Acceptance Criteria:**
- [ ] Order confirmation email to buyer within 60 seconds of payment; contains order ID, items, tracking URL
- [ ] Tracking URL accessible without login; HMAC token validates access
- [ ] Status progression displayed: Payment Confirmed → Processing → Shipped (with AWB and courier) → Delivered / Cancelled
- [ ] Buyer can cancel order in "Payment Confirmed" status; cancellation triggers Razorpay refund + cancellation email + stock restore
- [ ] Registered buyer can view all order history in account dashboard
- [ ] Guest order NOT linked to registered account created with same email (BR-018)

**Priority:** Must Have
**Dependencies:** AWS SES, Razorpay Refunds API, FR-ORDER-001 through FR-ORDER-005
**Applicable NFRs:** NFR-REL-006 (email delivery ≥ 99%), NFR-PE-001
**Applicable Business Rules:** BR-002 (cancellation window), BR-003 (no returns), BR-018 (guest data isolation)
**Feature Success Metric:** > 50% of buyers open the tracking URL at least once post-purchase; cancellation refund success rate > 99%
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Buyer-initiated return request (post-MVP), live carrier tracking API integration (post-MVP)

---

#### Feature 5.2 — Order Fulfillment Dashboard (Seller)

**One-line description:** Seller views, confirms, ships (with AWB), and marks orders delivered from the seller dashboard.

**User Job Addressed:** "I want to manage all my orders in one place instead of WhatsApp DMs." (OPP-003)

**User Stories:** US-007, US-008, US-009, US-033, US-034

**Acceptance Criteria:**
- [ ] Dashboard shows only the seller's own sub-orders; no cross-seller data visibility
- [ ] Orders sorted by most recent first; shows: order ID, buyer name (first + last initial), items, total, status
- [ ] Confirm → order moves to Processing; buyer's cancellation right revoked
- [ ] Ship: courier name + AWB number both required; → buyer receives "shipped" email within 60 seconds
- [ ] Mark Delivered available after Shipped
- [ ] Seller cannot view or update another seller's orders (HTTP 403/404)

**Priority:** Must Have
**Dependencies:** FR-FULFILL-001 through FR-FULFILL-006, AWS SES
**Applicable NFRs:** NFR-SEC-005 (multi-tenant isolation), NFR-PE-001
**Applicable Business Rules:** BR-002, BR-006 (confirmed orders eligible for settlement)
**Feature Success Metric:** > 95% of orders marked Shipped within 48 hours of confirmation; seller order dashboard daily active rate > 60% of active sellers
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Bulk order fulfillment, courier API integration, automated shipping label generation (post-MVP)

---

### Journey 6: Payout & Analytics

#### Feature 6.1 — Seller Payout Settlement

**One-line description:** Weekly automated settlement to seller bank accounts via Razorpay Payouts API; payout history visible in dashboard.

**User Job Addressed:** "I want regular cash flow from my online store without chasing payments." (OPP-005)

**User Stories:** US-024, US-032

**Acceptance Criteria:**
- [ ] Settlement ledger credits seller for each confirmed sub-order (net of Razorpay processing fee)
- [ ] Monday 09:00 IST automated payout job; disburses to each eligible seller's registered bank account via Razorpay Payouts (NEFT/IMPS)
- [ ] Payout record created: seller_id, amount, razorpay_payout_id, timestamp, status
- [ ] "Payout Initiated" email to seller within 60 seconds; amount + reference ID included
- [ ] Suspended sellers excluded; balance preserved for disbursement on reactivation (BR-013)
- [ ] Sellers can view payout history (last 12 months) in dashboard

**Priority:** Must Have
**Dependencies:** Razorpay Payouts API (account must have Payouts feature activated — OQ-002), AWS SES
**Applicable NFRs:** NFR-MAINT-005 (audit log for all payout events)
**Applicable Business Rules:** BR-005 (zero commission), BR-006 (weekly cycle), BR-013 (suspended withheld)
**Feature Success Metric:** 100% of eligible sellers receive payout within 24 hours of Monday 09:00 IST; payout failure rate < 2%
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** On-demand payout (seller-triggered), instant settlement (post-MVP)

---

#### Feature 6.2 — Seller Analytics Dashboard

**One-line description:** Basic sales dashboard showing revenue, order count, AOV (7d/30d), top 5 products, and inventory levels.

**User Job Addressed:** "I want to understand how my store is performing." (OPP-005)

**User Stories:** US-028, US-038

**Acceptance Criteria:**
- [ ] Revenue (INR), order count, AOV for last 7 days and last 30 days; scoped to seller
- [ ] Top 5 products by order count in last 30 days
- [ ] Export order data as CSV (RFC 4180, UTF-8): order ID, date, product, qty, price, buyer city, status

**Priority:** Should Have
**Dependencies:** FR-ANALYTICS-001, FR-ANALYTICS-002, FR-ANALYTICS-003
**Applicable NFRs:** NFR-PE-001
**Applicable Business Rules:** —
**Feature Success Metric:** Dashboard viewed by > 60% of active sellers at least once per week
**Prototype/Wireframe:** TBD — Phase 5 UX Design
**Out of Scope:** Real-time revenue graphs, cohort analysis, traffic sources, conversion funnel analytics (post-MVP)

---

## 9. Technical Architecture

### 9.1 System Architecture Overview

```
┌─────────────────── FRONTEND (Vercel / AWS) ────────────────────┐
│  Next.js 14 (App Router · TypeScript)                          │
│  SSR: product pages, category pages, homepage                  │
│  CSR: cart, checkout flow, seller dashboard, buyer dashboard   │
│  State: Zustand (cart, session)                                │
└──────────────────────────┬─────────────────────────────────────┘
                           │ HTTPS / REST /api/v1/
┌──────────────────────────▼─────────────────────────────────────┐
│  BACKEND (AWS ECS Fargate)                                     │
│  Django 5.x + DRF 3.15 (Python 3.12)                         │
│  Apps: sellers · products · buyers · cart · orders ·          │
│        payments · payouts · admin · notifications             │
│  Auth: RS256 JWT (djangorestframework-simplejwt)               │
└──┬────────────────────────┬──────────────────────────────────┬─┘
   │                        │                                  │
   ▼                        ▼                                  ▼
[PostgreSQL 16          [Redis 7                        [AWS S3]
 AWS RDS]               AWS ElastiCache]                Product images
 Primary DB             Cart · Sessions                 Store logo/banner
 UUID PKs               Product cache (TTL 1h)          [AWS CloudFront]
                        Cart TTL 24h                    Image CDN
   │
   ▼
[Razorpay]          [AWS SES]
Orders API          Transactional email
Subscriptions API   (order confirmation,
Payouts API         shipping, payout)
Webhooks
Refunds API
```

**Architecture principles:**
- **Django monolith** — not microservices; appropriate for MVP scale and 3-person team (CLAUDE.md)
- **UUID PKs throughout** — no integer IDs exposed externally
- **Managed services only** — no EC2 management; ECS Fargate + RDS + ElastiCache
- **Seller data isolation** — enforced at ORM layer via `seller_id` FK on all seller-scoped models; never at application logic layer
- **Async tasks** — email dispatch and payout jobs run as Celery tasks (AWS SQS or Redis-backed queue) to avoid blocking API responses

### 9.2 Integration Requirements Summary

| System | Type | Direction | Critical Dependency | SLA |
|--------|------|-----------|--------------------|----|
| Razorpay Orders API | REST | Outbound | Checkout cannot proceed without it | 99.99% |
| Razorpay Webhooks | HTTPS | Inbound | Orders not confirmed without it | — |
| Razorpay Subscriptions API | REST | Outbound | Seller billing not possible without it | 99.99% |
| Razorpay Payouts API | REST | Outbound | Weekly settlement not possible without it | 99.99% |
| Razorpay Refunds API | REST | Outbound | Order cancellation partial without it | 99.99% |
| AWS S3 | AWS SDK | Outbound | Product images not uploadable without it | 99.99% |
| AWS CloudFront | CDN | Delivery | Image delivery degraded; falls back to S3 origin | 99.9% |
| AWS SES | AWS SDK | Outbound | Email notifications unavailable; ops alert triggered | 99.9% |
| AWS ElastiCache Redis | SDK | Bidirectional | Cart degrades to DB fallback; sessions degraded | 99.9% |

*Full integration specifications: `docs/requirements/REQUIREMENTS.md §6` and `docs/requirements/USE-CASES.md`*

### 9.3 Core Data Model

| Entity | Key Fields | Relationships | Sensitivity |
|--------|-----------|---------------|-------------|
| Seller | id (UUID), email, store_name, status, subscription_status, gstin | → Products (1:N), → Orders/SubOrders (1:N), → Payouts (1:N) | PII + Financial |
| Product | id (UUID), seller_id (FK), name, slug, price, stock, status, category | → ProductImages (1:N), → OrderLineItems (1:N) | Public |
| Buyer | id (UUID), email, name, phone | → BuyerAddresses (1:N), → Orders (1:N) | PII |
| Order | id (UUID), buyer_id (FK nullable for guest), razorpay_order_id, total, status | → SubOrders (1:N), → OrderLineItems (1:N) | PII + Financial |
| SubOrder | id (UUID), order_id (FK), seller_id (FK), status, awb | → OrderLineItems (1:N) | Financial |
| OrderLineItem | id (UUID), sub_order_id (FK), product_id (FK), qty, unit_price | — | Financial |
| Subscription | id (UUID), seller_id (FK), razorpay_subscription_id, status, next_billing_date | — | Financial |
| SettlementLedger | id (UUID), seller_id (FK), sub_order_id (FK), amount, settled | — | Financial |
| Payout | id (UUID), seller_id (FK), amount, razorpay_payout_id, status, created_at | — | Financial |
| AuditLog | id (UUID), entity_type, entity_id, action, actor_id, timestamp, diff | — | Internal |

*Full data requirements: `docs/requirements/REQUIREMENTS.md §5`*

---

## 10. Analytics & Instrumentation Plan

> Analytics must be built in, not added after launch. All events tracked via a server-side analytics layer (backend-emitted events preferred for payment/order flows to prevent client-side manipulation).

### 10.1 Key Events Summary

| Event | Flow | Goal Metric |
|-------|------|-------------|
| `seller_registered` | Seller onboarding | Registration conversion |
| `store_setup_completed` | Seller onboarding | Setup completion rate (>80% target) |
| `subscription_payment_initiated` | Seller subscription | Funnel step |
| `subscription_payment_succeeded` | Seller subscription | Active seller count |
| `product_submitted_for_review` | Product listing | Listing funnel |
| `product_approved` / `product_rejected` | Product moderation | Admin throughput |
| `product_page_viewed` | Buyer browse | Discovery |
| `product_added_to_cart` | Cart | Cart conversion |
| `checkout_initiated` | Checkout | Checkout funnel start |
| `checkout_address_submitted` | Checkout | Form completion |
| `payment_initiated` | Payment | Payment funnel |
| `payment_succeeded` | Payment | Order creation; payment success rate |
| `payment_failed` | Payment | Checkout error rate |
| `order_placed` | Order | GMV, order count |
| `order_confirmed_by_seller` | Fulfillment | Seller activation |
| `order_shipped` | Fulfillment | Fulfillment rate |
| `order_delivered` | Fulfillment | Completion |
| `order_cancelled` | Cancellation | Cancellation rate |
| `payout_initiated` | Payout | Payout reliability |
| `seller_churned` | Retention | Churn rate |

*Full event taxonomy with property schemas: `docs/prd/PRD-ANALYTICS-PLAN.md`*

### 10.2 Core Funnels

**Seller Activation Funnel:**
`seller_registered` → `store_setup_completed` → `subscription_payment_succeeded` → `product_submitted_for_review` → `product_approved` → `order_placed` (first order)
*Target: > 70% of paid sellers receive first order within 30 days*

**Buyer Purchase Funnel:**
`product_page_viewed` → `product_added_to_cart` → `checkout_initiated` → `checkout_address_submitted` → `payment_initiated` → `payment_succeeded`
*Target: > 3% cart-to-order conversion*

**Guest Checkout Funnel:**
`checkout_initiated` (guest) → `checkout_address_submitted` → `payment_initiated` → `payment_succeeded`
*Target: > 60% of guest checkout initiations result in successful payment*

### 10.3 Privacy Compliance

India does not yet have GDPR-equivalent enforcement, but the Digital Personal Data Protection Act 2023 (DPDPA) applies. Events containing PII (buyer name, email, phone) SHALL NOT be sent to third-party analytics platforms without data processing agreements. Buyer and seller emails are stored in ShopNest's own database only — not exported to marketing platforms without explicit consent. *Full privacy mapping: `docs/prd/PRD-ANALYTICS-PLAN.md §5`*

---

## 11. Launch & Rollout Strategy

### 11.1 Feature Flag Strategy

For a 3-person MVP team, full feature-flag infrastructure is out of scope. Three targeted flags are defined for high-risk or easily-togglable components:

| Feature | Flag Name | Default | Rollout Plan |
|---------|----------|---------|-------------|
| Razorpay Payout disbursement (live) | `ff_payout_live` | Off (sandbox) | Off → internal test → On for first 5 sellers → 100% |
| Product approval required | `ff_product_admin_approval` | On | On at launch; can be toggled to Off for trusted-seller bypass post-MVP |
| Guest checkout | `ff_guest_checkout` | On | On at launch; Off is a fallback if abuse is detected |

All other features launch at 100% — no partial rollout required for MVP scope.

### 11.2 Staged Rollout Plan

| Stage | Audience | Criteria to Advance | Rollback Trigger |
|-------|----------|---------------------|-----------------|
| **Internal** | ShopNest team only | Full end-to-end transaction loop works; no P0 bugs after 48h | Any data loss or payment processing error |
| **Beta** | 10–25 invited sellers (beachhead cohort) + their buyers | Error rate < 1%; P95 < 200ms; ≥ 1 successful weekly payout completed | Checkout error rate > 2%; payout failure > 5% |
| **General Availability** | Public launch | 30-day beta: churn < 8%; NPS > 30; no P0 incidents | P0 incident (data loss, payment fraud, personal data breach) |

### 11.3 Launch Readiness Checklist

- [ ] All Must Have user stories passing acceptance tests
- [ ] Load test passing: 1,000 concurrent users, P95 API < 200ms, LCP < 2.5s
- [ ] Razorpay production account configured: Orders, Subscriptions, Payouts, Refunds APIs all tested
- [ ] AWS SES domain verified; DKIM configured; sending limits raised (production)
- [ ] AWS CloudWatch dashboards and alarms configured (uptime, error rate, latency)
- [ ] On-call rotation defined among 3 devs; P1 incident runbook documented
- [ ] GST subscription invoice: format reviewed by CA; GSTIN registered before first paying seller
- [ ] Consumer Protection (E-Commerce) Rules 2020 compliance reviewed by legal counsel
- [ ] `ff_payout_live` flag tested with ₹1 test payout to a team bank account
- [ ] Seller onboarding documentation (FAQ, help articles) published before beta
- [ ] Beta seller cohort (10–15 sellers) recruited and briefed on expectations

---

## 12. Constraints & Assumptions

### 12.1 Constraints

| ID | Type | Constraint | Impact |
|----|------|-----------|--------|
| CON-001 | Budget | AWS infrastructure ≤ ₹1,66,000/month (~$2,000/month) at MVP scale | Limits data volume and concurrent user capacity; inform infra decisions |
| CON-002 | Schedule | MVP delivery: 2026-10-31 | ~6.5 months from 2026-05-04; Phase 2–6 consume ~4 weeks |
| CON-003 | Team | 3 developers; no dedicated DevOps, QA, or support | All roles shared; managed services required; support load must be self-serve |
| CON-004 | Scope | India only; INR only | No multi-currency, no international shipping, no Stripe |
| CON-005 | Technical | Django monolith; no microservices | Appropriate for MVP; enforce modular app boundaries for post-MVP extraction |
| CON-006 | Technical | Razorpay only | Payout feature must be activated on ShopNest's account (OQ-002) |
| CON-007 | Legal | GST compliance required before first seller subscription | Engage CA pre-launch |
| CON-008 | Legal | Consumer Protection (E-Commerce) Rules 2020 | Marketplace operator registration and policy compliance required |
| CON-009 | Compliance | PCI-DSS delegated to Razorpay | Zero card data in ShopNest application layer; code audit required (Phase 10) |

### 12.2 Key Assumptions by Risk

| ID | Assumption | Risk if Wrong | Validation | Owner |
|----|-----------|--------------|------------|-------|
| A-01-016 | ₹1,999/month is acceptable subscription price | Sellers won't pay; MRR target missed | Beta pricing interviews (Q3 2026) | Product Owner |
| A-02-004 | Razorpay Payouts API available for ShopNest account | Weekly settlement not automatable | Confirm Razorpay account activation (OQ-002) | Tech Lead |
| A-01-002 | >70% buyer traffic is mobile | NFR targets set incorrectly for desktop | Measure from first 1,000 sessions post-launch | Full-stack Lead |
| A-02-002 | PostgreSQL FTS sufficient for ≤50K products | Search too slow; need Elasticsearch | Phase 9 load test (NFR-PE-005) | Backend dev |
| A-01-017 | AWS cost ≈ ₹1,66,000/month at 100 sellers | Budget exceeded at MVP scale | AWS Calculator validation before Phase 12 | Tech Lead |

*Full assumption catalog: `docs/assumptions/01-ideation-assumptions.md` and `docs/assumptions/02-requirements-assumptions.md`*

---

## 13. Open Questions

| ID | Question | Owner | Due Date | Status | Decision |
|----|----------|-------|----------|--------|---------|
| OQ-001 | Is ₹1,999/month the right subscription price? Single price or minor variants (e.g., ₹1,499 intro / ₹1,999 standard)? | Product Owner | Before beta launch (Q3 2026) | Open | — |
| OQ-002 | Has ShopNest's Razorpay account been activated for the Payouts feature? (Requires separate Razorpay onboarding and KYC.) | Tech Lead | Before Phase 7 Sprint 1 | Open | — |
| OQ-003 | What is the exact GST rate to apply to ShopNest's subscription invoice (18% standard SaaS rate assumed)? | Product Owner + CA | Before Phase 7 (subscription billing implementation) | Open | — |
| OQ-004 | Rejected seller re-registration: same email blocked permanently, or allow after 30 days? | Product Owner | Phase 4 (Architecture) | Open | — |
| OQ-005 | Is there a seller-facing category taxonomy (fixed list defined by platform) or do sellers define free-text categories? Phase 2 implies seller-defined; this affects search/browse taxonomy. | Product Owner | Phase 4 (Architecture) | Open | — |
| OQ-006 | Celery task queue for async email and payout jobs: Redis-backed (ElastiCache) or AWS SQS? | Tech Lead | Phase 4 (Architecture) | Open | — |

---

## 14. Glossary Reference

See `docs/prd/GLOSSARY.md` for all domain-specific terms used in this document.

---

## Appendix A: Referenced Documents

| Document | Path | Phase |
|----------|------|-------|
| Project Concept | `docs/ideation/PROJECT-CONCEPT.md` | Phase 1 |
| Feasibility Report | `docs/ideation/FEASIBILITY-REPORT.md` | Phase 1 |
| Stakeholder Map | `docs/ideation/STAKEHOLDER-MAP.md` | Phase 1 |
| Success Metrics | `docs/ideation/SUCCESS-METRICS.md` | Phase 1 |
| Market Sizing | `docs/ideation/MARKET-SIZING.md` | Phase 1 |
| Competitive Analysis | `docs/ideation/COMPETITIVE-ANALYSIS.md` | Phase 1 |
| Phase 1 Assumption Log | `docs/assumptions/01-ideation-assumptions.md` | Phase 1 |
| Requirements Specification | `docs/requirements/REQUIREMENTS.md` | Phase 2 |
| User Stories | `docs/requirements/USER-STORIES.md` | Phase 2 |
| Use Cases | `docs/requirements/USE-CASES.md` | Phase 2 |
| Business Rules | `docs/requirements/BUSINESS-RULES.md` | Phase 2 |
| Traceability Matrix | `docs/requirements/TRACEABILITY-MATRIX.md` | Phase 2 |
| Phase 2 Assumption Log | `docs/assumptions/02-requirements-assumptions.md` | Phase 2 |
| Phase 3 Assumption Log | `docs/assumptions/03-prd-assumptions.md` | Phase 3 |

---

*This PRD is the north star for Phases 4–14. All engineering, design, and delivery decisions should be evaluated against the goals, scope, and constraints documented here. Changes require a version increment and Product Owner + Tech Lead sign-off.*
