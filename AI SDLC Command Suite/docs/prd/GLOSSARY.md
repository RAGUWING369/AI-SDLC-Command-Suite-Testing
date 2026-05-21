# GLOSSARY — ShopNest
**Phase:** 03 — Product Requirements Document
**Generated:** 2026-05-05
**Status:** Draft — Awaiting Human Gate Approval
**Source Authority:** PRD.md §1–§14; REQUIREMENTS.md; BUSINESS-RULES.md

---

> **Purpose:** This glossary defines every domain-specific, business, and technical term used across ShopNest's Phase 1–3 artifacts. All downstream agents (Phase 4 onward) must use these definitions verbatim and must not redefine or rename these terms without raising a formal change request against this document.
>
> **Term selection criteria:** A term is included if it (a) has a specific ShopNest-defined meaning that differs from its general English meaning, (b) carries a precise business rule implication, or (c) is a technical abbreviation whose expansion is non-obvious to a new team member.

---

## A

**Active (Seller Account Status)**
A seller account state in which the seller has completed registration, received admin approval, and holds a current paid subscription — enabling their products to appear in the buyer-facing marketplace. Contrast: *Pending Review*, *Suspended*.

**Active (Product Status)**
A product listing state in which the product has received admin approval and belongs to a seller with Active account and Active subscription, making it visible and purchasable in the marketplace. Contrast: *Pending Review*, *Rejected*, *Delisted*.

**Active (Subscription Status)**
A subscription state in which the seller's recurring payment is current and has not lapsed; required for seller account and product visibility in the marketplace (see *Triple-Gate Visibility Rule*). Contrast: *Grace Period*, *Suspended*, *Cancelled*.

**Admin**
A ShopNest platform operator with elevated RBAC permissions to approve or reject seller accounts and product listings, suspend sellers, view platform-wide analytics, and manage all entities across the platform. Not to be confused with a *Seller* managing their own store. Synonym: *Platform Admin*.

**Assumption Log**
A per-phase artifact (stored under `docs/assumptions/`) that records every Tier 3 (high-confidence inference) the agent makes during a phase, enabling future validation and rollback if an inference is incorrect.

**AWB (Air Waybill)**
A carrier-issued tracking reference number that the seller manually enters into the ShopNest seller dashboard when shipping an order, enabling buyer order tracking. Carrier API integration is post-MVP; AWB entry is manual for MVP.

---

## B

**Beachhead Segment**
ShopNest's primary target customer group for initial market entry: fashion, lifestyle, and handmade-goods sellers in India's Tier 1 and Tier 2 cities who currently run their businesses via WhatsApp and spreadsheets and have 50–500 SKUs.

**BR (Business Rule)**
A formally defined, authority-backed constraint or policy governing platform behavior, identified by codes BR-001 through BR-018. Business rules take precedence over feature implementation decisions where they conflict.

**Buyer**
A person who browses the ShopNest marketplace, adds products to a cart, and completes a purchase via Razorpay. A buyer may be a *Guest Buyer* (no account required) or a *Registered Buyer* (account holder). Buyers do not have seller privileges.

**Buyer-Facing Marketplace**
The public-facing Next.js frontend (shopnest.in domain) through which buyers browse products, search, view product detail pages, and complete checkout. Distinguished from the *Seller Dashboard*.

---

## C

**Cart**
A temporary collection of product items a buyer intends to purchase, stored in Redis with a 24-hour TTL, keyed by session UUID (guest) or buyer UUID (registered). Cart state is lost after 24 hours of inactivity.

**Cart-to-Order Conversion Rate**
The percentage of buyer sessions that result in at least one completed order, measured as (orders placed ÷ sessions with at least one add-to-cart event) × 100. MVP target: >3%.

**Celery**
The distributed task queue used in the ShopNest Django backend to handle asynchronous operations including weekly payout job execution, email dispatch via AWS SES, and subscription renewal processing. Backed by Redis as the message broker.

**CloudFront**
AWS's CDN (Content Delivery Network) used to serve ShopNest product images and static assets from edge locations, targeting a ≤1-second load time for images ≤500 KB.

**CON (Constraint)**
A formally defined limitation on the project, identified by codes CON-001 through CON-010, that cannot be removed or deferred without a formal change to project scope (e.g., CON-001: AWS budget ≤ $2,000/month).

**Confirmed (Order Sub-status)**
The order state immediately after a seller explicitly acknowledges and accepts an order in the seller dashboard. Once an order reaches Confirmed status, a buyer can no longer cancel it (BR-002).

---

## D

**Daily Active Users (DAU)**
The count of unique users (buyers or sellers) who perform at least one action on the ShopNest platform within a single calendar day. Used as a platform health metric.

**Delivered (Order Sub-status)**
The final positive terminal state of an order sub-flow, set when the seller marks the sub-order as delivered in the seller dashboard after the buyer receives the shipment.

**Delisted (Product Status)**
A product that was previously Active but has been removed from marketplace visibility by an admin action or seller action, without the product record being deleted. Delisted products are invisible to buyers but retained for order history integrity.

**Django ORM**
The Object-Relational Mapper built into Django, used throughout the ShopNest backend to enforce multi-tenant seller data isolation by applying `seller_id` foreign key filters at the query level rather than at the database row-level security level.

**DPDPA (Digital Personal Data Protection Act)**
India's 2023 data protection legislation governing how ShopNest collects, processes, stores, and deletes personal data for Indian users. Relevant to analytics event collection (no PII in event properties), buyer data retention, and user account deletion requests.

---

## E

**Eligible Sub-order (Payout)**
A sub-order that qualifies for inclusion in a weekly payout batch: its payment status is `payment_captured = true` and its fulfillment status is one of `Processing`, `Shipped`, or `Delivered`. Sub-orders in `Cancelled` or `Refunded` states are excluded.

**EMI (Equated Monthly Instalment)**
A payment method available through Razorpay's checkout interface allowing buyers to pay for purchases in monthly instalments. EMI availability is determined by the buyer's bank; ShopNest does not gate or subsidise EMI — it is passed through via Razorpay's standard checkout options.

---

## F

**Feature Flag**
A runtime toggle controlling whether a given feature is active in the deployed environment. ShopNest MVP defines three feature flags: `ff_payout_live` (enables live Razorpay Payouts disbursement), `ff_product_admin_approval` (enables/disables admin approval gate for products), and `ff_guest_checkout` (enables/disables guest checkout flow).

**FR (Functional Requirement)**
A formally specified capability the system must provide, identified by codes FR-AREA-NNN (e.g., FR-CHECKOUT-005). Defined in `docs/requirements/REQUIREMENTS.md §2`. Total: 68 FRs across 12 functional areas.

**Fulfillment Status**
The lifecycle state of an individual sub-order from the seller's operational perspective: `Pending` → `Processing` → `Shipped` → `Delivered` (or `Cancelled`). Managed by the seller via the seller dashboard. Distinct from *Payment Status*.

---

## G

**GA (General Availability)**
The third and final rollout stage after Internal Testing and Beta, in which ShopNest is publicly available to all sellers and buyers without invite restriction.

**Grace Period**
A 7-day window (BR-007) immediately following a failed seller subscription payment during which the seller's account and products remain visible in the marketplace. If payment is not recovered by day 8, the seller account transitions to *Suspended*.

**GST (Goods and Services Tax)**
India's unified indirect tax, applicable to ShopNest's seller subscription invoices (18% GST on ₹1,999/month = ₹359.82 GST per invoice). ShopNest generates GST-compliant invoices for its own subscription billing only; seller-to-buyer B2C invoicing is outside ShopNest's scope for MVP.

**Guest Buyer**
A buyer who completes a purchase without creating a ShopNest account, providing name, email, phone, and delivery address at checkout. Guest orders are tracked via an HMAC-authenticated URL in the order confirmation email.

**GSTIN (Goods and Services Tax Identification Number)**
A 15-digit alphanumeric identifier issued to GST-registered businesses in India. Collected as an optional field during seller registration for MVP; format is captured but not validated against the GST portal API in MVP.

---

## H

**HMAC (Hash-based Message Authentication Code)**
A cryptographic technique used to generate tamper-proof, stateless authentication tokens for guest order tracking URLs. ShopNest uses HMAC-SHA256 derived from the order UUID and a server-side secret (stored in environment variables, never in the codebase).

**Human Gate**
A mandatory phase-completion checkpoint in the AI SDLC pipeline at which a stakeholder must explicitly reply `APPROVED` before the next phase begins. Human Gates prevent downstream work from proceeding on unapproved artifacts.

---

## I

**IMPS (Immediate Payment Service)**
An instant Indian interbank payment transfer method available 24/7. Used alongside NEFT as a transfer mode for weekly seller payouts via the Razorpay Payouts API.

**INR (Indian Rupee)**
The sole currency used across all ShopNest monetary values — product prices, seller subscriptions, platform payouts, and GST calculations. No multi-currency support in MVP.

**ISO/IEC 25010:2023**
The international standard for software product quality, defining 8 quality characteristics (Functional Suitability, Performance Efficiency, Compatibility, Usability, Reliability, Security, Maintainability, Portability) used as the NFR classification framework in ShopNest's REQUIREMENTS.md.

---

## J

**JWT (JSON Web Token)**
The stateless authentication token format used by ShopNest for user session management. Access tokens expire after 24 hours; refresh tokens expire after 30 days. Signed using RS256 (asymmetric RSA) rather than HS256 to support multi-service public key distribution.

---

## K

**KPI (Key Performance Indicator)**
A quantified business, user, or technical metric used to measure ShopNest's progress toward its desired outcomes. Defined in `docs/ideation/SUCCESS-METRICS.md`. Examples: MRR target ₹1,99,900 at 6 months; buyer cart-to-order conversion >3%.

---

## L

**LCP (Largest Contentful Paint)**
A Core Web Vitals metric measuring the time from page navigation to the point when the largest visible content element (typically the hero image on a product page) is fully rendered. ShopNest target: <2.5 seconds on mobile 4G (NFR-PE-002).

**Ledger Entry**
A credit record in the seller's settlement ledger table, created when an eligible sub-order is included in a payout batch. The ledger provides the authoritative record for seller payout history (FR-PAYOUT-006) and financial audit trails (BR-011).

---

## M

**Marketplace**
The buyer-facing side of the ShopNest platform — the browsable, searchable catalog of products from multiple independent sellers. The marketplace enforces *Triple-Gate Visibility*: only products from Active sellers with Active subscriptions and Active product status are displayed.

**MRR (Monthly Recurring Revenue)**
The predictable monthly revenue from all active seller subscriptions. Calculated as: active paying sellers × ₹1,999/month (ex-GST). Target: ₹1,99,900/month at 6-month mark (100 active sellers).

**Multi-tenant**
ShopNest's architectural pattern where multiple independent sellers share the same platform infrastructure, with strict data isolation enforced via `seller_id` foreign key constraints at the Django ORM level. No seller can access another seller's products, orders, or financial data.

**MVP (Minimum Viable Product)**
The initial ShopNest release scope targeting the core buyer + seller flow: seller onboarding and subscription, product listing and admin approval, buyer browsing and search, cart and checkout, order management and fulfillment, and weekly seller payouts. Features excluded from MVP are listed in PRD.md §5.2.

---

## N

**NEFT (National Electronic Funds Transfer)**
An Indian interbank electronic payment system used for batch fund transfers. Available 24/7 as of December 2019. Used alongside IMPS as a transfer mode for weekly seller payouts via the Razorpay Payouts API.

**NetBanking**
A payment method available through Razorpay's checkout allowing buyers to pay directly from their Indian bank account via the bank's online portal. One of the five Razorpay payment methods supported at ShopNest checkout.

**NFR (Non-Functional Requirement)**
A system quality attribute defined independently of specific features, classified using ISO/IEC 25010:2023. Identified by codes NFR-CATEGORY-NNN. Total: 37 NFRs across 8 categories. Defined in `docs/requirements/REQUIREMENTS.md §4`.

---

## O

**Order**
A top-level transactional record created when a buyer completes checkout, containing buyer details, delivery address, payment reference, and one or more *Sub-orders*. One order may span products from multiple sellers.

**OQ (Open Question)**
A formally tracked decision point identified during PRD synthesis that requires resolution before or during a specific downstream phase. Identified by codes OQ-001 through OQ-006 in PRD.md §13.

**ORM (Object-Relational Mapper)**
See *Django ORM*.

---

## P

**P95 (95th Percentile)**
A latency measurement representing the response time experienced by the slowest 5% of requests in a load test. Used for all ShopNest performance NFRs (e.g., NFR-PE-001: API P95 < 200ms).

**Payment Captured**
The Razorpay webhook event (`payment.captured`) confirming that a buyer's payment has been successfully authorized and settled to ShopNest's Razorpay account. This event triggers: stock decrement (BR-008), order creation, payout ledger credit eligibility, and order confirmation emails.

**Payment Confirmation**
See *Payment Captured*.

**Payment Status**
The lifecycle state of an order's payment from Razorpay's perspective: `Pending` → `Authorized` → `Captured` (or `Failed` / `Refunded`). Distinct from *Fulfillment Status*.

**PCI-DSS (Payment Card Industry Data Security Standard)**
The security compliance framework for handling cardholder data. ShopNest delegates all PCI-DSS compliance to Razorpay by using Razorpay's hosted checkout — ShopNest never receives, processes, stores, or transmits raw card data.

**Pending Review (Seller Account Status)**
The initial seller account state after registration, awaiting admin approval. A seller in Pending Review cannot publish products or access the full seller dashboard.

**Pending Review (Product Status)**
The initial product listing state after a seller submits a new product, awaiting admin approval before it can appear in the marketplace. Products in Pending Review are invisible to buyers.

**Platform Admin**
See *Admin*.

**PostgreSQL FTS (Full-Text Search)**
PostgreSQL's built-in full-text search capability used for ShopNest's product search feature at MVP scale (≤50,000 products). Sufficient for P95 ≤500ms at MVP scale; Elasticsearch is a post-MVP upgrade path when search volume or quality demands it.

**Processing (Order Sub-status)**
The order sub-status set when the seller confirms a sub-order in the seller dashboard, indicating that the order has been accepted and is being prepared for shipment.

---

## R

**RBAC (Role-Based Access Control)**
ShopNest's authorization model defining three user roles — *Buyer*, *Seller*, and *Platform Admin* — each with a distinct set of permitted API endpoints and dashboard actions. Role assignment is stored in the User entity and enforced by Django REST Framework permission classes.

**Razorpay Orders API**
The Razorpay API used to create payment orders for buyer checkout flows, generating a Razorpay order ID that is passed to the Razorpay checkout widget on the frontend.

**Razorpay Payouts API**
The Razorpay API used to disburse weekly seller settlements to Indian bank accounts via NEFT or IMPS. Requires separate Payouts feature activation on ShopNest's Razorpay account (distinct from standard payment processing onboarding).

**Razorpay Refunds API**
The Razorpay API used to initiate refunds to buyers' original payment methods when a buyer cancels an order that is still in a cancellable state (before seller confirmation, BR-002).

**Razorpay Subscriptions API**
The Razorpay API used to manage seller monthly subscription billing, handling recurring charge attempts, invoice generation, failure retry, and webhook events (`subscription.charged` / `subscription.charge.failed`).

**Razorpay Webhooks**
HTTP POST callbacks from Razorpay to ShopNest's webhook endpoint, confirming payment events (`payment.captured`, `payment.failed`), subscription events (`subscription.charged`, `subscription.charge.failed`), and refund events. All webhooks are validated using HMAC-SHA256 signature verification (NFR-SEC-006).

**Registered Buyer**
A buyer who has created a ShopNest account and is authenticated at checkout, enabling order history access, saved addresses, and persistent cart across sessions. Contrast: *Guest Buyer*.

**Rejected (Product Status)**
A product listing state set by an admin when a product fails the approval review, with a mandatory rejection reason provided to the seller. A rejected product can be edited and resubmitted by the seller.

**Rejected (Seller Account Status)**
A seller account state set by an admin when a seller registration application is denied. The seller's email address is blocked from re-registration; the seller must contact ShopNest support to appeal.

**RS256**
An asymmetric JWT signing algorithm (RSA + SHA-256) using a private key to sign tokens and a public key to verify them. Preferred over HS256 for ShopNest's multi-service architecture because the public key can be distributed to services without exposing the signing secret.

**RTO / RPO (Recovery Time Objective / Recovery Point Objective)**
Disaster recovery targets. ShopNest targets RTO < 1 hour and RPO < 1 hour, meaning the platform must be restored within 1 hour of a failure and data loss must not exceed 1 hour of transactions.

---

## S

**S3 (AWS Simple Storage Service)**
AWS object storage used to store all ShopNest product images uploaded by sellers. Images are never stored in the PostgreSQL database. CloudFront CDN serves images from S3 to buyers.

**SaaS (Software as a Service)**
ShopNest's delivery model: a hosted multi-vendor marketplace platform that sellers access via a monthly subscription, without downloading or installing software.

**SAM (Serviceable Addressable Market)**
The subset of TAM that ShopNest can realistically target given its constraints: ~850,000 Indian online sellers with annual marketable revenue of ~₹2,039 crore, derived from the full TAM filtered by internet access, 50–500 SKU range, and India-only geography.

**SES (AWS Simple Email Service)**
The AWS-managed transactional email service used by ShopNest to send all platform emails: order confirmations, shipping notifications, cancellation confirmations, new order alerts to sellers, and payout notifications. No SMS or WhatsApp for MVP.

**Seller**
An independent business owner or store manager who registers on ShopNest, pays a monthly subscription, lists products via the seller dashboard, and fulfills orders placed by buyers. Each seller is an isolated tenant on the platform.

**Seller Dashboard**
The authenticated web interface (Next.js client-side rendered) through which sellers manage their store profile, product listings, incoming orders, fulfillment status, and payout history. Access requires Active seller account status with Active subscription.

**Settlement Ledger**
The PostgreSQL table recording all payout-eligible credits and completed disbursements per seller. Forms the basis of seller payout history (FR-PAYOUT-006) and is subject to 7-year financial record retention (BR-011).

**SKU (Stock Keeping Unit)**
A distinct product variant or item in a seller's inventory, uniquely identified by a combination of product attributes (e.g., colour, size). For ShopNest's beachhead segment, target sellers manage 50–500 SKUs.

**SOM (Serviceable Obtainable Market)**
The realistic portion of SAM ShopNest can capture in Year 1: ~500 paying sellers / ₹59.97 lakh annual revenue (base case), representing approximately 0.06% of SAM.

**SSG (Static Site Generation)**
A Next.js rendering strategy pre-building pages at build time for maximum performance. Used for ShopNest's category landing pages (SEO-critical, content changes infrequently).

**SSR (Server-Side Rendering)**
A Next.js rendering strategy generating HTML on the server at request time. Used for ShopNest's product listing pages and product detail pages (SEO-critical, dynamic content, must reflect current availability and pricing).

**Sub-order**
The portion of an Order attributed to a single seller, containing only that seller's products. A single buyer Order may generate multiple sub-orders if the cart contains products from different sellers. Sub-orders are the unit of fulfillment and payout settlement.

**Suspended (Seller Account Status)**
The seller account state triggered when a seller's subscription payment is not recovered within the 7-day grace period (BR-007). Suspended sellers' products are delisted from the marketplace; the seller retains read-only access to historical data. Payouts are withheld during suspension.

---

## T

**TAM (Total Addressable Market)**
The maximum theoretical market opportunity if ShopNest captured 100% of the target market: ~15.1 million Indian online sellers / ₹3.62 lakh crore annual revenue (based on IBEF and MoSPI data, labelled as industry benchmark — not project-specific).

**Triple-Gate Visibility Rule**
ShopNest's BR-001 policy governing marketplace product visibility: a product is shown to buyers only when ALL THREE conditions are simultaneously true — (1) seller account status = Active, (2) seller subscription status = Active, and (3) product status = Active. Failure of any gate removes the product from the marketplace.

**TTL (Time to Live)**
The expiration duration for cached data. ShopNest Redis TTL values: cart data = 24 hours; product catalog cache = 1 hour; sessions = per JWT expiry.

---

## U

**UC (Use Case)**
A detailed specification of a user interaction with the platform, including actor, preconditions, main flow, alternative flows, and postconditions. Identified by codes UC-001 through UC-006. Defined in `docs/requirements/USE-CASES.md`.

**UPI (Unified Payments Interface)**
India's real-time payment system enabling instant bank-to-bank transfers via mobile number or VPA (Virtual Payment Address). One of the five Razorpay payment methods supported at ShopNest checkout; the dominant mobile payment method in India.

**US (User Story)**
A requirement expressed from an end-user perspective following the format "As a [persona], I want to [action] so that [benefit]." Identified by codes US-001 through US-038. Defined in `docs/requirements/USER-STORIES.md`.

**UUID (Universally Unique Identifier)**
A 128-bit identifier used as the primary key for all ShopNest database entities, ensuring globally unique IDs without sequential enumeration. PostgreSQL's `gen_random_uuid()` function generates UUIDs. Using UUIDs (not integer IDs) is a hard constraint for the ShopNest data model.

---

## V

**VPA (Virtual Payment Address)**
A UPI identifier (e.g., `name@bankname`) used to receive funds. Required for seller bank account records to enable Razorpay Payouts API disbursement via UPI. Collected during seller store setup alongside IFSC code and account number.

---

## W

**Webhook**
See *Razorpay Webhooks*.

**Weekly Payout**
ShopNest's seller disbursement cycle (BR-006): every Monday at 09:00 IST, an automated Celery job aggregates all eligible sub-orders from the preceding week and initiates bank transfers via the Razorpay Payouts API. Sellers with suspended accounts are excluded from the payout run.

---

## Z

**Zero Commission**
ShopNest's core business model commitment (BR-005): the platform charges no percentage-based fee on seller transactions. Revenue comes exclusively from seller subscriptions. Sellers receive 100% of their product revenue net of the Razorpay payment gateway processing fee (~2% + GST), which ShopNest treats as a platform operating cost.

---

*This glossary is the authoritative source for all ShopNest terminology.*
*Phase 4 (Architecture) and downstream agents must not introduce synonyms or rename these terms without a formal change request.*
*Last updated: 2026-05-05 (Phase 3 — PRD)*
