# Business Rules Catalog — ShopNest
**Phase:** 02 — Requirements Engineering
**Standard:** BABOK v3 — Business Rules Analysis
**Generated:** 2026-05-04
**Status:** Draft — Awaiting Human Gate Approval

---

> **Why a separate catalog:** Business rules are domain-level constraints that govern behavior regardless of which software feature implements them. They are stable across multiple releases and missing them is a leading cause of production defects. Every business rule in this catalog is traceable to an authority.

---

## BR-001 — Seller Marketplace Visibility (Triple-Gate Rule)

**Rule:** A seller's products SHALL NOT appear in the buyer-facing marketplace unless ALL three of the following conditions are simultaneously true:
1. Seller account status = **Active** (approved by Platform Admin)
2. Seller subscription status = **Active** (subscription payment current)
3. Each individual product status = **Active** (approved by Platform Admin)

If any one condition becomes false, the seller's products are immediately hidden from buyers. The seller's account is not deleted; data is preserved.

**Authority:** ShopNest platform moderation and subscription policy  
**Downstream FRs:** FR-PRODUCT-003, FR-SUBSCR-005, FR-ADMIN-004  
**Traceability:** US-003, US-005, US-022, US-023

---

## BR-002 — Order Cancellation Window (Buyer)

**Rule:** A buyer MAY cancel an order only while the order status is "Payment Confirmed" (i.e., before the assigned seller confirms it). Once the seller transitions the order to "Processing" status, the buyer's cancellation right is permanently forfeited for that order.

**Authority:** ShopNest MVP cancellation policy — confirmed Phase 2 gap scan (2026-05-04)  
**Downstream FRs:** FR-ORDER-003, FR-FULFILL-002  
**Traceability:** US-020

---

## BR-003 — No Product Returns in MVP

**Rule:** ShopNest does not accept or process product return requests or return-based refunds in the MVP release. The only buyer-initiated refund mechanism is order cancellation under BR-002. All sales are final once a seller confirms an order.

**Authority:** ShopNest MVP scope decision (Phase 1 PROJECT-CONCEPT.md — Out of Scope)  
**Downstream FRs:** FR-ORDER-003  
**Traceability:** US-020

---

## BR-004 — Single Currency (INR Only)

**Rule:** All prices, order amounts, subscription fees, payout amounts, and invoice values on the platform SHALL be denominated exclusively in Indian Rupees (INR). The system does not accept, display, or convert any foreign currency.

**Authority:** ShopNest India-only geographic scope decision (Phase 1 gap scan)  
**Downstream FRs:** FR-CHECKOUT-003, FR-PAYOUT-002  
**Traceability:** All monetary user stories

---

## BR-005 — Zero Per-Transaction Commission

**Rule:** ShopNest charges sellers a flat monthly subscription fee only. No per-transaction fee, GMV commission, or percentage-based charge is deducted from buyer order payments. Sellers receive 100% of the buyer payment amount, net of Razorpay's payment processing fee (which Razorpay deducts from ShopNest's collected amount before settlement).

**Clarification:** The Razorpay processing fee (typically 2% + GST for domestic transactions) is borne by ShopNest from its subscription revenue margin, not passed on to sellers separately in MVP. This is a business model decision to be revisited at 500+ sellers if margin is pressured.

**Authority:** ShopNest zero-commission business model (Phase 1 gap scan)  
**Downstream FRs:** FR-PAYOUT-001  
**Traceability:** US-024

---

## BR-006 — Weekly Payout Settlement Cycle

**Rule:** Seller payout disbursements execute on a fixed weekly cycle, every **Monday at 09:00 IST**. Only sub-orders in status "Processing", "Shipped", or "Delivered" are eligible for inclusion in the current week's settlement (i.e., sub-orders that the seller has confirmed). Sub-orders still in "Payment Confirmed" status (buyer cancellation window open) are NOT included in the settlement until confirmed by the seller.

A seller with a settlement balance of ₹0 receives no payout entry that week. The minimum disbursable amount is ₹1.

**Authority:** ShopNest payout policy — confirmed Phase 2 gap scan (2026-05-04)  
**Downstream FRs:** FR-PAYOUT-001, FR-PAYOUT-002  
**Traceability:** US-024

---

## BR-007 — Subscription Lapse and Grace Period

**Rule:**
1. A first subscription payment failure transitions the seller to "Payment Failed" status.
2. A 7-day grace period begins from the payment failure date. During the grace period, the seller retains full dashboard and product-listing access.
3. If no successful payment is received within the 7-day grace period, the seller transitions to "Suspended" status: storefront and all products are hidden from buyers; seller dashboard is locked.
4. A successful subscription renewal at any point — including during the grace period or after suspension — immediately restores the seller to "Active" status and re-publishes their products.
5. Seller data (products, orders, customers) is NEVER deleted due to subscription lapse. It is preserved and becomes accessible again upon reactivation.

**Authority:** ShopNest subscription policy — accepted Phase 2 gap scan (Tier 2 suggestion confirmed)  
**Downstream FRs:** FR-SUBSCR-003, FR-SUBSCR-004, FR-SUBSCR-005, FR-SUBSCR-006  
**Traceability:** US-003

---

## BR-008 — Stock Decrement at Payment Confirmation

**Rule:** Product stock quantities SHALL be decremented at the moment of order creation (triggered by Razorpay `payment.captured` webhook), not when the seller confirms the order. This prevents overselling when multiple buyers purchase the same item concurrently.

If a stock decrement would result in a negative quantity (concurrent purchases exceeding stock), the later order SHALL fail with an "out of stock" error. The stock decrement operation SHALL be atomic (database transaction with SELECT FOR UPDATE or equivalent).

**Authority:** Inventory management best practice — prevents overselling  
**Downstream FRs:** FR-CHECKOUT-008, FR-CHECKOUT-012  
**Traceability:** US-017

---

## BR-009 — Product Deletion with Active Orders

**Rule:** A product with one or more orders in status other than "Delivered" or "Cancelled" SHALL NOT be permanently deleted. Sellers may deactivate/archive the product (hide from new buyers) but the product record must be preserved to support ongoing order fulfillment. Platform admins follow the same rule. Deletion is permitted only when all referencing orders are in a terminal state.

**Authority:** Data integrity and order fulfillment continuity policy  
**Downstream FRs:** FR-PRODUCT-011  
**Traceability:** US-004

---

## BR-010 — GST Invoicing Scope

**Rule:** ShopNest generates GST-compliant tax invoices exclusively for its own subscription billing transactions (ShopNest → Seller for the monthly subscription fee). ShopNest does NOT generate tax invoices for seller-to-buyer product purchase transactions. Sellers are solely responsible for issuing GST-compliant invoices to their own customers under their own GSTIN registration.

**Authority:** Indian GST Act (CGST Act 2017, Section 31) + Phase 2 gap scan (confirmed 2026-05-04)  
**Downstream FRs:** FR-SUBSCR-007  
**Traceability:** US-036

---

## BR-011 — Financial Record Retention (7-Year Minimum)

**Rule:** All financial records — including orders, order line items, payments, refunds, subscriptions, settlement ledger entries, payout disbursements, and GST invoices — SHALL be retained for a minimum of 7 years from the date of the transaction. Records SHALL NOT be permanently deleted before this retention window expires, regardless of account closure.

**Authority:** Indian Companies Act 2013 (Section 128 — books of accounts for 8 years); GST Act 2017 (books for 6 years, Section 35); conservative 7-year policy adopted  
**Downstream FRs:** Data Requirements §5.1  
**Traceability:** All financial user stories

---

## BR-012 — Seller Self-Purchase Prohibition

**Rule:** A seller account MAY NOT place a purchase order as a buyer for products in their own store within the same session. The system SHALL detect and reject checkout attempts where the buyer's authenticated account UUID matches the seller UUID of any product in the cart.

**Authority:** Platform integrity and review manipulation prevention policy  
**Downstream FRs:** FR-CHECKOUT-006  
**Traceability:** US-016, US-017

---

## BR-013 — Payout Withheld During Suspension

**Rule:** Weekly payout disbursements SHALL NOT be executed for sellers with subscription status "Suspended". The accumulated settlement balance for a suspended seller is preserved in the ledger and disbursed in the first payout cycle after the seller's subscription is restored to "Active" status.

**Authority:** ShopNest payout-subscription linkage policy  
**Downstream FRs:** FR-PAYOUT-005  
**Traceability:** US-024

---

## BR-014 — Platform Admin Account Provisioning

**Rule:** Platform Admin accounts SHALL only be created through a direct database seeding process or by an existing Platform Admin via the admin panel. There is no public-facing registration path for the Platform Admin role. A user cannot elevate their own role to Platform Admin through any API endpoint.

**Authority:** Platform security policy — least privilege principle  
**Downstream FRs:** FR-AUTH-011, FR-ADMIN-001  
**Traceability:** US-021

---

## BR-015 — Product Image Constraints

**Rule:** Each product listing MAY contain between 1 and 5 images. Each image file must: (a) be in JPEG or PNG format, (b) not exceed 5MB in file size. No other file formats are accepted. These constraints apply to both product images and store logo/banner images (store logo max 2MB, banner max 5MB).

**Authority:** AWS S3 storage cost management + platform security (prevents executable upload)  
**Downstream FRs:** FR-PRODUCT-002, FR-SELLER-005  
**Traceability:** US-004

---

## BR-016 — Buyer Shipping Address PII Handling

**Rule:** A buyer's shipping address, phone number, and full name collected during checkout are classified as Personally Identifiable Information (PII). This data SHALL be accessible only to: (a) the seller fulfilling that specific order, (b) the buying account holder (registered buyers only), and (c) Platform Admins. It SHALL NOT be exposed to any other seller or any unauthenticated endpoint.

**Authority:** Consumer Protection (E-Commerce) Rules 2020; Information Technology (Reasonable Security Practices) Rules 2011  
**Downstream FRs:** FR-FULFILL-001, NFR-SEC-005  
**Traceability:** US-016, US-027

---

## BR-017 — Product Approval Re-Trigger Conditions

**Rule:** Editing a product's **price** or **stock quantity** does NOT require re-approval by the Platform Admin; the product remains in "Active" status immediately after saving.

Editing a product's **name**, **description**, or **images** transitions the product back to "Pending Review" status and hides it from buyers until re-approved by a Platform Admin.

**Authority:** Moderation workflow efficiency policy — prevents admin overload from trivial edits while ensuring material content changes are reviewed  
**Downstream FRs:** FR-PRODUCT-008  
**Traceability:** US-006

---

## BR-018 — Guest Buyer Data Linkage

**Rule:** Guest checkout data (name, email, phone, address) is associated with an order UUID only — not with any buyer account. If the guest later creates a registered buyer account with the same email, the historical guest orders are NOT automatically linked to the new buyer account in MVP. Guest order history is accessible only via the order tracking URL.

**Authority:** MVP simplicity — guest-to-account linking is a post-MVP feature  
**Downstream FRs:** FR-ORDER-001, FR-ORDER-005  
**Traceability:** US-016, US-025

---

*Sources: CLAUDE.md; Phase 1 ideation artifacts; Phase 2 gap scan answers (2026-05-04)*  
*All rules cross-referenced to REQUIREMENTS.md Section 3 and TRACEABILITY-MATRIX.md.*
