# Requirements Specification — ShopNest
**Phase:** 02 — Requirements Engineering
**Standard:** ISO/IEC/IEEE 29148:2018 · ISO/IEC 25010:2023
**Generated:** 2026-05-04
**Status:** Draft — Awaiting Human Gate Approval

## Document Control
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-05-04 | Requirements Agent (Phase 2) | Initial specification |

---

## 1. System Context

### 1.1 Context Diagram

```
External Actors                      ShopNest Platform                 External Systems
──────────────                       ─────────────────                 ────────────────

[Seller]
  ──registration, store/product data──▶
  ──order status updates (AWB)────────▶
  ──subscription payment────────────▶  ┌─────────────────────────┐
  ◀──order alerts, invoices, payouts──  │                         │  ──payment init/subscription──▶ [Razorpay API]
                                        │   ShopNest Platform     │  ◀──payment webhooks (success/failure/subscription)──
[Buyer (Guest)]                         │   (Next.js + Django)    │
  ──browse/search, cart, checkout data─▶│                         │  ──image upload──────────────▶ [AWS S3]
  ◀──catalog, order confirmation email─  │                         │  ◀──image CDN delivery────────────────────────────
                                        │                         │
[Buyer (Registered)]                    │                         │  ──payout disbursements──────▶ [Razorpay Payouts API]
  ──all guest actions + login, history─▶│                         │
  ◀──all guest responses + order history│                         │  ──transactional emails──────▶ [AWS SES]
                                        │                         │  ◀──delivery status callbacks──
[Platform Admin]                        │                         │
  ──seller/product approvals, suspense─▶│                         │  ──cache read/write──────────▶ [AWS ElastiCache Redis]
  ◀──pending queues, platform metrics──  │                         │
                                        └─────────────────────────┘
                                                    │
                                               [PostgreSQL 16
                                                AWS RDS]
```

### 1.2 System Boundary

**In-boundary (ShopNest owns these):**
- Seller registration, onboarding, subscription management
- Product listing, moderation, and catalogue
- Shopping cart, checkout orchestration
- Order lifecycle management (creation → fulfillment → delivery)
- Seller payout settlement ledger
- Buyer account management
- Platform admin panel
- Transactional email dispatch (via AWS SES)
- Product image management (upload to S3, serve via CloudFront)

**Out-of-boundary (delegated to external systems):**
- Payment processing and PCI-DSS compliance → Razorpay
- Payout bank transfer execution → Razorpay Payouts API
- Email delivery infrastructure → AWS SES
- Physical order delivery → seller-managed logistics
- Seller-to-buyer GST invoicing → seller's responsibility

---

## 2. Functional Requirements

Modal verb rules:
- **SHALL** — mandatory; deviation is a defect
- **SHOULD** — recommended; deviation must be documented
- **MAY** — optional capability

---

### FR-AUTH — Authentication & Authorization

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-AUTH-001 | The system SHALL allow sellers to register with: email address, password, full name, business name, store name, phone (10-digit Indian mobile), city, GSTIN (optional), and shipping origin pincode (6-digit). | Must Have | US-001 | POST /api/v1/sellers/register with valid payload returns 201; duplicate email returns 409 |
| FR-AUTH-002 | The system SHALL reject seller registration if the email address is already associated with any account in the system, returning HTTP 409 with an error message. | Must Have | US-001 | Duplicate email → 409 response; original account unaffected |
| FR-AUTH-003 | The system SHALL hash all passwords using bcrypt with a cost factor of 12 or higher before storage. | Must Have | US-001 | Database inspection confirms no plaintext password; hash prefix is `$2b$12$` |
| FR-AUTH-004 | The system SHALL issue a JWT access token (expiry: 24 hours) and a refresh token (expiry: 30 days) upon successful authentication for both sellers and registered buyers. | Must Have | US-001, US-025 | Successful login response contains `access_token` and `refresh_token`; decoded JWT exp matches 24h |
| FR-AUTH-005 | The system SHALL allow registered buyers to create accounts with: email address, password, full name, and phone (10-digit Indian mobile). | Should Have | US-025 | POST /api/v1/buyers/register with valid payload returns 201 |
| FR-AUTH-006 | The system SHALL implement role-based access control (RBAC) with three roles: **Buyer**, **Seller**, and **Platform Admin**. Each role SHALL have access only to its designated endpoints. | Must Have | US-001, US-025 | Seller JWT rejected on buyer-only endpoints; buyer JWT rejected on seller dashboard endpoints |
| FR-AUTH-007 | The system SHALL prevent access to all Seller Dashboard endpoints for sellers whose subscription status is not "Active". | Must Have | US-003 | Seller with Payment Failed subscription receives HTTP 403 on dashboard endpoints |
| FR-AUTH-008 | The system SHALL prevent access to all Platform Admin endpoints unless the request carries a valid Platform Admin role JWT. | Must Have | US-021 | Admin JWT accepted on /api/v1/admin/; Seller JWT rejected on same endpoints → 403 |
| FR-AUTH-009 | The system SHALL enforce a minimum password length of 8 characters, with at least 1 numeric character, for all account types. | Must Have | US-001 | Password "abc123" (6 chars) → 400; "abcd1234" (8 chars, 1 digit) → 201 |
| FR-AUTH-010 | The system SHOULD invalidate the refresh token upon explicit logout by the user. | Should Have | US-001 | POST /api/v1/auth/logout with valid refresh token → 200; subsequent refresh with same token → 401 |
| FR-AUTH-011 | Platform Admin accounts SHALL be created only via a secure administrative process; self-registration as Platform Admin SHALL be rejected. | Must Have | US-021 | POST /api/v1/sellers/register with role=admin payload → 400 or role ignored |

---

### FR-SELLER — Seller Registration & Store Management

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-SELLER-001 | The system SHALL present a guided store setup wizard to newly registered sellers, collecting: store name (max 100 chars), store logo (PNG/JPG, max 2MB), banner image (PNG/JPG, max 5MB), and up to 5 product categories. | Must Have | US-002 | Seller completing wizard with valid data → store record created; seller proceeds to subscription step |
| FR-SELLER-002 | The system SHALL prevent a newly registered seller from accessing the product listing interface until the store setup wizard is completed AND a subscription payment is successful. | Must Have | US-002 | GET /api/v1/seller/products before wizard completion → 403 |
| FR-SELLER-003 | The system SHALL initiate the seller subscription payment flow via Razorpay Subscriptions API immediately after the store setup wizard is completed. | Must Have | US-003 | Wizard completion → Razorpay subscription created; seller redirected to Razorpay checkout |
| FR-SELLER-004 | The system SHALL capture and store seller payout bank account details (account holder name, account number, IFSC code) after successful subscription signup. | Must Have | US-003 | Bank details saved; not shown in API responses except to the owning seller |
| FR-SELLER-005 | The system SHALL upload store logo and banner images to AWS S3 and serve them via CloudFront CDN. | Must Have | US-002 | Image URLs in store response begin with CloudFront domain |
| FR-SELLER-006 | The system SHALL allow sellers with an Active subscription to update their store name, logo, banner, categories, and shipping origin pincode at any time. | Should Have | US-031 | PATCH /api/v1/seller/store with valid data → 200; changes reflected in marketplace |
| FR-SELLER-007 | The system SHALL allow sellers to update their payout bank account details independently of their subscription status. | Could Have | US-037 | PATCH /api/v1/seller/payout-details → 200; existing pending payouts not affected |

---

### FR-PRODUCT — Product Management

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-PRODUCT-001 | The system SHALL allow sellers to create a product listing with: name (max 200 chars), description (max 2,000 chars), price (INR, min ₹1, max 2 decimal places), stock quantity (integer ≥ 0), category (from seller's declared store categories), and between 1 and 5 product images. | Must Have | US-004 | POST /api/v1/seller/products with valid payload → 201; status = "Pending Review" |
| FR-PRODUCT-002 | The system SHALL upload product images to AWS S3 (validated MIME type: image/jpeg or image/png; max 5MB per image) and serve via CloudFront CDN. | Must Have | US-004 | Image with MIME type application/pdf → 400; valid PNG ≤ 5MB → stored at S3 URL |
| FR-PRODUCT-003 | The system SHALL assign every newly submitted product a status of "Pending Review" and SHALL NOT display it in the buyer-facing marketplace until a Platform Admin sets it to "Active". | Must Have | US-005 | GET /marketplace/products returns only Active products; Pending Review product absent |
| FR-PRODUCT-004 | The system SHALL add new product submissions to the Platform Admin's Product Approval Queue with: product name, seller store name, submission timestamp, and image thumbnails. | Must Have | US-022 | Admin dashboard shows queue entry within 5 seconds of product submission |
| FR-PRODUCT-005 | The system SHALL allow Platform Admins to approve a product, transitioning its status from "Pending Review" to "Active", and SHALL send an approval email to the seller. | Must Have | US-022 | POST /api/v1/admin/products/{id}/approve → 200; product appears in marketplace; seller receives email |
| FR-PRODUCT-006 | The system SHALL allow Platform Admins to reject a product, requiring a rejection reason (1–500 chars), transitioning its status to "Rejected", and SHALL send a rejection email to the seller including the reason. | Must Have | US-030 | POST /api/v1/admin/products/{id}/reject with reason → 200; product absent from marketplace; seller email received with reason |
| FR-PRODUCT-007 | The system SHALL allow sellers to edit a product's name, description, price, stock quantity, category, and images at any time the product is not deleted. | Must Have | US-006 | PATCH /api/v1/seller/products/{id} → 200; changes persisted |
| FR-PRODUCT-008 | The system SHALL re-submit a product to "Pending Review" status ONLY when the seller modifies the product name, description, or images. Changes to price or stock quantity SHALL NOT trigger re-review. | Must Have | US-006 | Price change → product remains Active; name change → product transitions to Pending Review |
| FR-PRODUCT-009 | The system SHALL automatically set a product's status to "Out of Stock" and hide it from buyer search and browse when its stock quantity reaches 0. | Must Have | US-015 | Stock set to 0 → product absent from category page and search results |
| FR-PRODUCT-010 | The system SHALL send an in-dashboard notification to the seller when any of their product's stock quantities falls to 5 or below. | Must Have | US-010 | Stock updated to 5 → low-stock flag appears in seller dashboard within 60 seconds |
| FR-PRODUCT-011 | The system SHALL allow sellers to delete a product that has no active orders referencing it; the system SHALL reject deletion of products with orders in status other than "Delivered" or "Cancelled". | Must Have | US-004 | Delete product with active order → 409; delete product with all-Delivered orders → 204 |
| FR-PRODUCT-012 | The system SHALL allow Platform Admins to unpublish any Active product at any time, transitioning it to "Admin Removed" status, with a mandatory reason, and SHALL notify the seller by email. | Must Have | US-023 | POST /api/v1/admin/products/{id}/remove → 200; product disappears from marketplace; seller email sent |

---

### FR-BROWSE — Marketplace Browsing & Search

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-BROWSE-001 | The system SHALL render the marketplace homepage using server-side rendering (SSR), displaying the 20 most recently approved products and all product categories. | Must Have | US-011 | GET / returns fully rendered HTML with product data; Googlebot crawler sees product content |
| FR-BROWSE-002 | The system SHALL render product category pages using SSR, displaying all Active products in the selected category, paginated at 24 products per page. | Must Have | US-012 | GET /category/{slug}?page=2 returns correct product set; HTML is fully rendered |
| FR-BROWSE-003 | The system SHALL render product detail pages using SSR, displaying: product name, all images (carousel), price (INR), description, seller store name, stock status, and an "Add to Cart" button. | Must Have | US-013 | GET /products/{slug} returns fully rendered HTML; og:title and og:description meta tags present |
| FR-BROWSE-004 | The system SHALL display "Out of Stock" on product detail pages and disable the "Add to Cart" button when stock quantity is 0. | Must Have | US-013 | Out-of-stock product detail page: button disabled; POST to add-to-cart → 409 |
| FR-BROWSE-005 | The system SHALL provide keyword search across all Active product names and descriptions using PostgreSQL full-text search, returning results sorted by relevance score, paginated at 24 per page. | Must Have | US-014 | GET /api/v1/search?q=cotton+shirt returns products with those terms in name/description |
| FR-BROWSE-006 | The system SHALL return search results within 500ms at P95 for queries against a catalog of up to 50,000 active products under standard load. | Must Have | US-014 | Load test: 100 concurrent search requests → P95 ≤ 500ms |
| FR-BROWSE-007 | The system SHALL generate stable, SEO-friendly URL slugs for product pages (e.g., /products/blue-cotton-kurti-{uuid-prefix}) that do not change when the product name is edited. | Must Have | US-013 | Product name edit → URL unchanged; old URL still resolves to correct product |

---

### FR-CART — Shopping Cart

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-CART-001 | The system SHALL allow any visitor (guest or authenticated buyer) to add an Active, in-stock product to a cart without requiring authentication. | Must Have | US-015 | POST /api/v1/cart/items without auth token → 201; cart session cookie set |
| FR-CART-002 | The system SHALL persist guest cart data in Redis with a TTL of 24 hours from the last modification. | Must Have | US-015 | Cart created; 24h later (simulated TTL expiry) → cart data gone |
| FR-CART-003 | The system SHALL persist authenticated buyer cart data in Redis associated with the buyer's UUID, with a TTL of 24 hours from the last modification. | Should Have | US-026 | Buyer adds to cart, logs out, logs in → cart restored |
| FR-CART-004 | The system SHALL allow buyers to update item quantity (min 1, max stock quantity) or remove items from the cart. | Must Have | US-015 | PATCH /api/v1/cart/items/{id} with qty > stock → 409; qty = 0 treated as remove |
| FR-CART-005 | The system SHALL validate product stock availability at checkout initiation; if any item is out of stock, the system SHALL return an error listing the affected products and remove them from the cart before presenting checkout. | Must Have | US-016 | Checkout with item that sold out → 409 with product name; item removed from cart |
| FR-CART-006 | The system SHALL display the running cart subtotal (sum of item price × quantity) in INR, updated in real time as the buyer modifies the cart. | Must Have | US-015 | Add 2 items → subtotal = sum of (price × qty); remove one → subtotal decreases |
| FR-CART-007 | The system SHALL support a cart containing Active products from multiple sellers simultaneously. | Must Have | US-015 | Cart with products from Seller A and Seller B → single checkout flow; split into sub-orders |

---

### FR-CHECKOUT — Checkout & Payment

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-CHECKOUT-001 | The system SHALL present a guest checkout form collecting: full name, email address, phone (10-digit Indian mobile), and shipping address (address line 1, city, state, pincode). | Must Have | US-016 | Checkout form submitted with invalid phone (9 digits) → 400; valid data → proceeds |
| FR-CHECKOUT-002 | The system SHALL allow authenticated buyers to select a saved address or enter a new address at checkout. | Should Have | US-026 | Registered buyer checkout: saved address pre-populated; "Use new address" option available |
| FR-CHECKOUT-003 | The system SHALL create a Razorpay Order for the total cart amount (INR, in paise) via the Razorpay Orders API before presenting the payment interface to the buyer. | Must Have | US-017 | Razorpay Order ID created and returned; amount matches cart total |
| FR-CHECKOUT-004 | The system SHALL present the Razorpay checkout interface supporting: UPI (UPI ID and QR), credit card, debit card, and NetBanking payment methods. | Must Have | US-017 | Razorpay checkout modal loads with all four payment method tabs visible |
| FR-CHECKOUT-005 | The system SHALL confirm payment success exclusively via Razorpay webhook signature validation (HMAC-SHA256); client-side payment callbacks SHALL NOT be used as the sole payment confirmation mechanism. | Must Have | US-017 | Forged client-side success callback without webhook → order NOT created; payment reconciliation log shows mismatch |
| FR-CHECKOUT-006 | The system SHALL create a platform Order record with status "Payment Confirmed" upon receiving and validating a `payment.captured` webhook from Razorpay. | Must Have | US-017 | Simulate Razorpay payment.captured webhook → Order created in DB with correct status |
| FR-CHECKOUT-007 | The system SHALL split a multi-seller cart into seller-specific sub-orders, each linked to the parent order and its respective seller. | Must Have | US-017 | Cart with 2 sellers → 1 Order record + 2 sub-order records; each seller sees only their sub-order |
| FR-CHECKOUT-008 | The system SHALL decrement each ordered product's stock quantity atomically upon order creation, preventing negative stock. | Must Have | US-017 | Concurrent checkout of last unit → only one order succeeds; second returns 409 |
| FR-CHECKOUT-009 | The system SHALL send an order confirmation email to the buyer within 60 seconds of a successful `payment.captured` webhook, containing: order ID, itemized list, total amount, and seller contact info. | Must Have | US-018 | Payment confirmed → buyer email delivered within 60 seconds (checked via SES delivery log) |
| FR-CHECKOUT-010 | The system SHALL send a new-order notification email to each seller whose products are included in the order, within 60 seconds of successful payment. | Must Have | US-007 | Payment confirmed → each seller email delivered within 60 seconds |
| FR-CHECKOUT-011 | The system SHALL NOT store, log, or transmit raw card numbers, CVVs, or card expiry dates at any point in the application layer. | Must Have | US-017 | Code review + WAF log audit: no card data in application logs or database |
| FR-CHECKOUT-012 | The system SHALL handle `payment.failed` webhooks by marking the pending order as "Payment Failed" and restoring the decremented stock quantities. | Must Have | US-017 | Simulate payment.failed webhook → stock restored; order marked Payment Failed |

---

### FR-ORDER — Order Management (Buyer-Side)

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-ORDER-001 | The system SHALL generate a unique, publicly accessible order tracking URL containing the order UUID and a single-use HMAC verification token for guest buyers. | Must Have | US-019 | Order tracking URL in confirmation email → accessible without login; shows correct status |
| FR-ORDER-002 | The system SHALL display the current order status and AWB number (when provided by seller) on the order tracking page. | Must Have | US-019 | Seller marks Shipped with AWB "123456" → tracking page shows "Shipped" + AWB |
| FR-ORDER-003 | The system SHALL allow a buyer to cancel an order that is in "Payment Confirmed" status, triggering a full refund via Razorpay Refunds API and transitioning the order to "Cancelled". | Must Have | US-020 | POST /api/v1/orders/{id}/cancel on Payment Confirmed order → 200; Razorpay refund initiated; stock restored |
| FR-ORDER-004 | The system SHALL send a cancellation confirmation email to the buyer upon successful order cancellation, including the refund amount and expected credit timeline (Razorpay standard: 5–7 business days). | Must Have | US-020 | Cancellation confirmed → buyer receives email within 60 seconds |
| FR-ORDER-005 | Authenticated buyers SHALL be able to view all their historical orders, including status, items, and total amount, in their account dashboard. | Should Have | US-027 | GET /api/v1/buyer/orders → returns all orders for authenticated buyer; guest orders not included |

---

### FR-FULFILL — Order Fulfillment (Seller-Side)

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-FULFILL-001 | The system SHALL display all sub-orders assigned to a seller in their dashboard, sorted by creation date (most recent first), with: order ID, buyer name (first name + last initial), product list, quantity, total amount, and current status. | Must Have | US-007 | Seller dashboard shows only their own sub-orders; does not show other sellers' orders |
| FR-FULFILL-002 | The system SHALL allow sellers to confirm a sub-order (transition from "Payment Confirmed" to "Processing"), after which the buyer can no longer cancel. | Must Have | US-008 | POST /api/v1/seller/orders/{id}/confirm → 200; buyer cancel attempt on same order → 409 |
| FR-FULFILL-003 | The system SHALL allow sellers to mark a sub-order as "Shipped" by providing a courier name and AWB number (both required fields, AWB max 50 chars). | Must Have | US-009 | POST /api/v1/seller/orders/{id}/ship with courier_name and awb → 200; buyer tracking page updated |
| FR-FULFILL-004 | The system SHALL send a "Your order has been shipped" email to the buyer upon seller marking an order as "Shipped", including the courier name and AWB number. | Must Have | US-033 | Seller marks Shipped → buyer email delivered within 60 seconds |
| FR-FULFILL-005 | The system SHALL allow sellers to mark a sub-order as "Delivered". | Should Have | US-034 | POST /api/v1/seller/orders/{id}/deliver → 200; order status updated |
| FR-FULFILL-006 | The system SHALL prevent any seller from viewing, confirming, or updating sub-orders that belong to a different seller. | Must Have | US-007 | Seller A JWT + Seller B's order ID → 403 or 404 |

---

### FR-NOTIF — Notifications

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-NOTIF-001 | The system SHALL send all transactional notifications exclusively via email using AWS Simple Email Service (SES). No SMS or push notification channels in MVP. | Must Have | US-018 | No Twilio/SMS calls observed in application logs; SES send events logged per notification |
| FR-NOTIF-002 | The system SHALL deliver the following buyer notification emails: Order Confirmation, Order Shipped (with AWB), Order Cancelled (with refund details). | Must Have | US-018, US-020, US-033 | Each event → corresponding email received; content verified against template |
| FR-NOTIF-003 | The system SHALL deliver the following seller notification emails: New Order Received, Product Approved, Product Rejected (with reason), Payout Initiated, Low-Stock Alert. | Must Have | US-007, US-005, US-030 | Each event → corresponding email received |
| FR-NOTIF-004 | The system SHALL retry failed email sends up to 3 times with exponential backoff; after 3 failures, the failure SHALL be logged with event_type, entity_id, and error reason for operations review. | Must Have | US-018 | SES failure simulated → 3 retry attempts logged; final failure entry in error log |

---

### FR-PAYOUT — Seller Payouts

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-PAYOUT-001 | The system SHALL maintain a settlement ledger per seller, crediting the full received order amount (net of Razorpay payment processing fee) for each sub-order that transitions to "Processing" status or beyond. | Must Have | US-024 | Sub-order confirmed → settlement ledger entry created with correct net amount |
| FR-PAYOUT-002 | The system SHALL execute a weekly bulk payout job every Monday at 09:00 IST, disbursing the accumulated settlement balance to each seller's registered bank account via the Razorpay Payouts API (NEFT or IMPS). | Must Have | US-024 | Payout job runs Monday 09:00 IST; Razorpay Payout API called for each eligible seller |
| FR-PAYOUT-003 | The system SHALL record each payout disbursement with: seller UUID, amount (INR), Razorpay payout reference ID, initiation timestamp, and status (Initiated / Completed / Failed). | Must Have | US-024 | Payout record in DB with all required fields after disbursement |
| FR-PAYOUT-004 | The system SHALL send a "Payout Initiated" email to the seller upon payout initiation, including the amount and Razorpay payout reference ID. | Must Have | US-024 | Payout initiated → seller email received within 60 seconds |
| FR-PAYOUT-005 | The system SHALL NOT disburse payouts to sellers with subscription status "Suspended". | Must Have | US-024 | Suspended seller with non-zero balance → payout job skips them; balance retained |
| FR-PAYOUT-006 | Sellers SHALL be able to view their payout history (last 12 months of disbursements) from the seller dashboard. | Should Have | US-032 | GET /api/v1/seller/payouts → returns list sorted by date descending |

---

### FR-SUBSCR — Seller Subscription Management

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-SUBSCR-001 | The system SHALL create a Razorpay Subscription (monthly billing cycle) for each seller upon successful first payment during onboarding. | Must Have | US-003 | Razorpay subscription object created; seller subscription_status = Active in DB |
| FR-SUBSCR-002 | The system SHALL display each seller's subscription status (Active / Payment Failed / Suspended), current plan name, and next billing date in the seller dashboard. | Should Have | US-036 | Dashboard shows correct status and next billing date pulled from Razorpay subscription |
| FR-SUBSCR-003 | The system SHALL transition a seller's subscription status to "Payment Failed" upon receiving a `subscription.charge.failed` webhook from Razorpay and send the seller a payment failure email. | Must Have | US-003 | Simulate charge.failed webhook → seller status = Payment Failed; email delivered |
| FR-SUBSCR-004 | The system SHALL grant sellers full dashboard and product-listing access during a 7-day grace period after a payment failure. | Must Have | US-003 | Seller with Payment Failed status (day 3) → dashboard accessible; day 8 → 403 |
| FR-SUBSCR-005 | The system SHALL transition a seller's status to "Suspended" and hide their storefront and all products from the marketplace after the 7-day grace period expires without successful payment. | Must Have | US-003 | Day 8 post-failure → seller products absent from marketplace; seller receives suspension email |
| FR-SUBSCR-006 | The system SHALL restore a seller's status to "Active" and re-publish their products upon receiving a `subscription.charged` (success) webhook after a previous failure. | Must Have | US-003 | Simulate successful renewal webhook after suspension → seller status = Active; products visible |
| FR-SUBSCR-007 | The system SHALL generate a GST-compliant PDF subscription invoice for each successful subscription payment and make it downloadable from the seller dashboard. | Must Have | US-036 | Successful billing → PDF invoice available at /api/v1/seller/invoices/{id}/download |

---

### FR-ADMIN — Platform Administration

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-ADMIN-001 | The system SHALL display a Seller Approval Queue showing all sellers with "Pending Review" status, with: store name, registration date, email, and phone. | Must Have | US-021 | New seller registration → appears in admin queue within 5 seconds |
| FR-ADMIN-002 | The system SHALL allow Platform Admins to approve a seller, transitioning status to "Active" and sending an approval email to the seller. | Must Have | US-021 | POST /api/v1/admin/sellers/{id}/approve → 200; seller receives email; can access dashboard |
| FR-ADMIN-003 | The system SHALL allow Platform Admins to reject a seller application with a mandatory reason (1–500 chars), sending a rejection email to the applicant. | Should Have | US-029 | POST /api/v1/admin/sellers/{id}/reject with reason → 200; applicant receives rejection email |
| FR-ADMIN-004 | The system SHALL allow Platform Admins to suspend an Active seller account, hiding their storefront and all products and sending a suspension email. | Must Have | US-023 | POST /api/v1/admin/sellers/{id}/suspend → 200; seller products absent from marketplace |
| FR-ADMIN-005 | The system SHALL allow Platform Admins to reactivate a suspended seller account, restoring their storefront and products. | Must Have | US-023 | POST /api/v1/admin/sellers/{id}/reactivate → 200; seller products visible again |
| FR-ADMIN-006 | The system SHALL display a platform overview dashboard showing: total active sellers, total active products, total orders (last 7 days / last 30 days), and total GMV (last 7 days / last 30 days). | Could Have | US-035 | GET /api/v1/admin/dashboard → returns all six metrics with correct values |

---

### FR-ANALYTICS — Seller Analytics

| ID | Requirement | Priority | User Story | Test Condition |
|----|-------------|----------|------------|----------------|
| FR-ANALYTICS-001 | The system SHALL display in the seller dashboard: total revenue (INR), total order count, and average order value for the last 7 days and last 30 days. | Should Have | US-028 | Seller with 5 orders (₹1,000 each) in last 7 days → revenue = ₹5,000; count = 5; AOV = ₹1,000 |
| FR-ANALYTICS-002 | The system SHALL display the seller's top 5 products by order count in the last 30 days. | Should Have | US-028 | Products ranked by order count; ties broken by product name alphabetically |
| FR-ANALYTICS-003 | Sellers SHALL be able to export their order data in CSV format, including: order ID, date, product name, quantity, unit price, buyer city, and order status. | Could Have | US-038 | GET /api/v1/seller/orders/export → CSV file download with correct columns and data |

---

## 3. Business Rules

*(Full catalog in `docs/requirements/BUSINESS-RULES.md`)*

| ID | Business Rule | Authority |
|----|--------------|-----------|
| BR-001 | A seller's products SHALL NOT appear in the marketplace unless: seller status = Active AND subscription status = Active AND each product status = Active. All three conditions must be true simultaneously. | ShopNest platform policy |
| BR-002 | A buyer MAY NOT cancel an order that has been confirmed (status = "Processing") by the seller. | ShopNest MVP cancellation policy (Phase 2 gap scan) |
| BR-003 | No product returns or return-refund requests are processed in the MVP. The only buyer-initiated refund mechanism is order cancellation before seller confirmation. | ShopNest MVP scope decision |
| BR-004 | All monetary amounts on the platform are denominated in Indian Rupees (INR). Foreign currency pricing is not supported. | ShopNest India-only scope decision |
| BR-005 | ShopNest charges zero per-transaction commission on buyer purchases. Seller revenue is subscription fees only. | ShopNest zero-commission business model |
| BR-006 | Seller settlement payouts execute every Monday at 09:00 IST. Eligible orders are those in "Processing", "Shipped", or "Delivered" status with a confirmed sub-order. | ShopNest payout policy |
| BR-007 | A subscription payment failure triggers a 7-day grace period. After 7 days without successful renewal, the seller account is Suspended. | ShopNest subscription policy |
| BR-008 | Stock quantity is decremented at order creation (payment confirmed), not at seller confirmation. Negative stock is never permitted. | Inventory management — prevents overselling |
| BR-009 | A product with active orders (status not Delivered or Cancelled) SHALL NOT be deleted by the seller or removed by Platform Admin without first cancelling those orders. | Data integrity policy |
| BR-010 | GST invoices are generated by ShopNest for subscription billing only. ShopNest does not generate seller-to-buyer invoices. | Indian GST Act + Phase 2 gap scan |
| BR-011 | Order, payment, and invoice records SHALL be retained for a minimum of 7 years. | Indian Companies Act / GST record-keeping requirement |
| BR-012 | A seller MAY NOT purchase from their own store as a buyer in the same session. | Platform integrity policy |
| BR-013 | Seller payout is withheld for sellers with subscription status "Suspended". Held balances are disbursed when the seller's subscription is restored to "Active". | ShopNest payout-subscription linkage policy |
| BR-014 | Platform Admin accounts are created by existing admins only; no self-registration path for the Admin role exists. | Platform security policy |
| BR-015 | Product images: maximum 5 images per product; maximum 5MB per image; accepted formats: JPEG, PNG only. | Storage cost and security policy |

---

## 4. Non-Functional Requirements

### 4.1 Performance Efficiency (ISO 25010:2023)

| ID | Requirement | Metric | Target | Measurement Method |
|----|-------------|--------|--------|--------------------|
| NFR-PE-001 | API response time — all endpoints | P95 latency | < 200ms | Load test at 80% peak load (800 concurrent users); AWS CloudWatch |
| NFR-PE-002 | Product page Largest Contentful Paint | LCP on mobile 4G (375px) | < 2.5s | Core Web Vitals via Google PageSpeed Insights |
| NFR-PE-003 | First Contentful Paint (buyer-facing pages) | FCP | < 1.5s | Core Web Vitals |
| NFR-PE-004 | Product image load time | Via CloudFront CDN, image ≤ 500KB | < 1.0s | CloudFront access logs + browser timing |
| NFR-PE-005 | Marketplace search response time | P95 at ≤ 50,000 active products | < 500ms | Load test with concurrent search queries |
| NFR-PE-006 | Concurrent user capacity | Users transacting simultaneously without degradation | ≥ 1,000 | Locust / k6 load test at 1,000 concurrent users; P95 latency within NFR-PE-001 |
| NFR-PE-007 | Checkout completion time (end-to-end) | Add to cart → payment confirmation displayed | < 3 minutes under normal load | Manual test + timing in Razorpay sandbox |
| NFR-PE-008 | Database query latency | P95 across all queries | < 50ms | AWS RDS Performance Insights |

### 4.2 Reliability (ISO 25010:2023)

| ID | Requirement | Metric | Target | Measurement Method |
|----|-------------|--------|--------|--------------------|
| NFR-REL-001 | Platform uptime | Monthly uptime % | ≥ 99.9% (< 43.8 min downtime/month) | AWS CloudWatch uptime alarm |
| NFR-REL-002 | Recovery Time Objective (RTO) | Max downtime after critical incident | < 1 hour | Incident runbook + drill |
| NFR-REL-003 | Recovery Point Objective (RPO) | Max data loss window | < 1 hour | AWS RDS automated backup frequency (every 1 hour) |
| NFR-REL-004 | Mean Time to Recovery (MTTR) | Average time to restore service after P1 incident | < 2 hours | Post-incident review log |
| NFR-REL-005 | Payment webhook idempotency | Duplicate Razorpay webhook delivery | Zero duplicate orders or stock decrements | Simulate duplicate webhook delivery; assert single order in DB |
| NFR-REL-006 | Email notification delivery rate | Transactional emails successfully delivered | ≥ 99% | AWS SES delivery metrics |

### 4.3 Security (ISO 25010:2023)

| ID | Requirement | Metric | Target | Measurement Method |
|----|-------------|--------|--------|--------------------|
| NFR-SEC-001 | Data in transit | TLS version | TLS 1.2 minimum; TLS 1.3 preferred | SSL Labs scan on production domain |
| NFR-SEC-002 | Data at rest (RDS) | Encryption | AES-256 via AWS RDS encryption | AWS Console: encryption enabled on RDS instance |
| NFR-SEC-003 | Authentication | Token standard | RS256-signed JWT; access: 24h; refresh: 30d | Decoded JWT: alg=RS256; exp field verified |
| NFR-SEC-004 | Password storage | Hashing algorithm | bcrypt, cost factor ≥ 12 | DB inspection: hash prefix = `$2b$12$` |
| NFR-SEC-005 | Multi-tenant data isolation | Seller data cross-access | Zero cross-seller data leaks | Automated test: Seller A token → Seller B endpoints → 403/404 |
| NFR-SEC-006 | Payment webhook validation | HMAC-SHA256 signature | 100% of webhooks validated before processing | Unit test: invalid signature → webhook rejected |
| NFR-SEC-007 | Card data handling | PCI-DSS scope | ShopNest application stores zero card data | Code audit + WAF log review: no card data patterns in logs |
| NFR-SEC-008 | Input validation | OWASP Top 10 | Zero SQL injection, XSS, command injection vectors | SAST scan (Phase 10 Security Review) |
| NFR-SEC-009 | File upload security | Accepted MIME types | JPEG and PNG only; max 5MB | Upload executable → 400; oversized → 400 |
| NFR-SEC-010 | Access control | Admin endpoint protection | Admin endpoints return 403 for non-admin roles | Automated RBAC test suite |

### 4.4 Usability — Interaction Capability (ISO 25010:2023)

| ID | Requirement | Metric | Target | Measurement Method |
|----|-------------|--------|--------|--------------------|
| NFR-USA-001 | Mobile responsiveness — buyer | Minimum viewport width | ≥ 375px fully functional | Manual test on iPhone SE (375px) and Samsung Galaxy A (360px) |
| NFR-USA-002 | Tablet/desktop responsiveness — seller | Minimum viewport width | ≥ 768px fully functional | Manual test on iPad (768px) |
| NFR-USA-003 | Accessibility | WCAG standard | WCAG 2.1 Level AA for all buyer-facing pages | Axe DevTools automated scan + manual keyboard navigation test |
| NFR-USA-004 | Seller time to first product listed | From signup to first published product (pending review) | < 30 minutes | Usability test with 5 representative sellers (post-beta) |
| NFR-USA-005 | Browser support | Supported browser versions | Chrome 120+, Safari 17+, Firefox 120+, Samsung Internet 23+; iOS 16+, Android 12+ | Browserstack cross-browser test on all listed browsers |
| NFR-USA-006 | Language | Interface language | English only (MVP) | Manual review: no non-English strings in UI |

### 4.5 Maintainability (ISO 25010:2023)

| ID | Requirement | Metric | Target | Measurement Method |
|----|-------------|--------|--------|--------------------|
| NFR-MAINT-001 | Test coverage — backend (Django) | Line coverage | ≥ 80% | pytest-cov report in CI pipeline |
| NFR-MAINT-002 | Test coverage — frontend (Next.js) | Line coverage | ≥ 80% | Jest + Istanbul coverage report in CI pipeline |
| NFR-MAINT-003 | API documentation | OpenAPI spec | 100% of /api/v1/ endpoints documented in openapi.yaml | Contract test: all endpoints in code present in spec |
| NFR-MAINT-004 | Logging format | Structured logging | 100% of application log entries are JSON with fields: timestamp, level, service, event_type, request_id, user_id (hashed) | Log aggregation check in CloudWatch |
| NFR-MAINT-005 | Audit trail | Auditable events | All money-affecting and account-status-affecting actions produce an audit log entry | Audit log populated for: order creation, payment events, subscription events, payout events, admin actions |
| NFR-MAINT-006 | Infrastructure as code | Configuration management | 100% of infrastructure defined in Docker Compose (local) and ECS Task Definitions (production) | No manual server configuration; all config in code |

### 4.6 Compatibility — Interoperability (ISO 25010:2023)

| ID | Requirement | Metric | Target | Measurement Method |
|----|-------------|--------|--------|--------------------|
| NFR-COMPAT-001 | API style | REST convention | All endpoints follow RESTful conventions at /api/v1/ with snake_case JSON | API design review against OpenAPI spec |
| NFR-COMPAT-002 | Razorpay API version | Compatibility | Razorpay API version 2024-01 or later | Razorpay API version header in all requests |
| NFR-COMPAT-003 | Product image URL stability | CDN URL persistence | Active product CloudFront URLs do not change or expire | URL sampled at product approval; unchanged after 30 days |
| NFR-COMPAT-004 | Data export format | CSV export | Seller order export uses RFC 4180 CSV format with UTF-8 encoding | Export file opened in Excel without encoding errors |

### 4.7 Portability (ISO 25010:2023)

| ID | Requirement | Metric | Target | Measurement Method |
|----|-------------|--------|--------|--------------------|
| NFR-PORT-001 | Containerization | OCI compliance | All application components packaged as OCI-compliant Docker images | docker build + docker run succeeds from clean environment |
| NFR-PORT-002 | Infrastructure | Managed services only | No EC2 instance management; all compute via ECS Fargate | AWS Console: zero EC2 instances in production; compute = Fargate tasks |
| NFR-PORT-003 | Business logic portability | WSGI compatibility | Django application runnable on any WSGI-compatible host (Gunicorn, uWSGI) | Local Gunicorn run succeeds without changes to application code |

### 4.8 Functional Suitability (ISO 25010:2023)

Validated via functional requirements (Section 2), acceptance criteria (USER-STORIES.md), and use case specifications (USE-CASES.md). Full test coverage defined in Phase 9 (Testing).

| ID | Requirement | Target | Measurement Method |
|----|-------------|--------|--------------------|
| NFR-FS-001 | Functional completeness | All Must Have user stories implemented and passing acceptance tests | Phase 9 test results: 100% Must Have story pass rate |
| NFR-FS-002 | Data correctness | Zero incorrect financial calculations | Automated test: order total = sum(price × qty) for all orders; payout = sum of sub-order amounts |

---

## 5. Data Requirements

### 5.1 Key Entity Register

| Entity | Description | Owner | Sensitivity | Retention Period | Volume (Launch / 12mo) |
|--------|-------------|-------|-------------|-----------------|------------------------|
| Seller Account | Store owner profile, credentials, subscription status | Seller | PII (email, phone, GSTIN, bank details) | 7 years post-account closure | 100 / 500 |
| Buyer Account | Consumer profile and credentials | Buyer | PII (email, phone, address) | 7 years post-last-order | 500 / 5,000 |
| Guest Checkout Info | Name, email, phone, address for guest orders | Platform | PII | 7 years post-order date | 100 / 2,000 |
| Product | Product listing, metadata, pricing, stock | Seller | Public | 7 years post-deletion | 5,000 / 50,000 |
| Product Image | S3 object key, CloudFront URL | Seller | Public | Same as parent product | 25,000 / 250,000 |
| Order | Purchase transaction header | Platform | PII + Financial | 7 years (GST/legal) | 1,000 / 20,000 |
| Order Line Item | Product × quantity × price per order | Platform | Financial | 7 years | 3,000 / 60,000 |
| Cart | Active shopping session (Redis) | Buyer/Guest | Session (PII in guest checkout step) | 24 hours (Redis TTL) | N/A (in-memory) |
| Subscription | Seller Razorpay subscription record | Platform | Financial | 7 years | 100 / 500 |
| Settlement Ledger Entry | Per-order credit to seller | Platform | Financial | 7 years | 1,000 / 20,000 |
| Payout | Weekly disbursement record | Platform | Financial | 7 years | 50 / 2,600 |
| Invoice (GST) | Subscription invoice PDF | Platform | Financial | 7 years | 100 / 500 |
| Audit Log | System action trail (money, account, admin events) | Platform | Internal | 7 years | 10,000 / 500,000 |

### 5.2 Data Export Requirements

| Export | Format | Trigger | Consumer |
|--------|--------|---------|----------|
| Seller order data | RFC 4180 CSV, UTF-8 | On-demand by seller | Seller (business records) |
| Platform financial summary | N/A (admin dashboard display only in MVP) | — | Platform Admin |

### 5.3 Analytics / Reporting Data

| Report | Data Required | Access | Update Frequency |
|--------|--------------|--------|-----------------|
| Seller sales summary (7d/30d) | Orders, amounts, product counts scoped to seller | Seller dashboard | Near real-time (on page load) |
| Top 5 products by order count | Order line items scoped to seller | Seller dashboard | Daily |
| Platform overview metrics | Aggregated sellers, products, orders, GMV | Admin dashboard | Near real-time |

---

## 6. Integration Requirements

| System | Integration Type | Direction | Data Exchanged | Authentication | Availability SLA | Error Handling Strategy |
|--------|-----------------|-----------|---------------|----------------|-----------------|------------------------|
| Razorpay Orders API | REST API | Outbound | Order creation (amount_in_paise, currency=INR, receipt=order_uuid, notes) | API key (header) | 99.99% | Retry ×3 with exponential backoff (1s, 2s, 4s); surface error to buyer after all retries; log full error |
| Razorpay Webhooks | HTTPS POST | Inbound | payment.captured, payment.failed, subscription.charged, subscription.charge.failed | HMAC-SHA256 (X-Razorpay-Signature header) | — | Idempotency key on all webhook handlers; retry up to 3 times on processing failure; alert ops on consecutive failures |
| Razorpay Subscriptions API | REST API | Outbound | Subscription creation (plan, customer, total_count, notify_info) | API key | 99.99% | Retry ×3; if all fail → flag seller for manual resolution + notify ops |
| Razorpay Payouts API | REST API | Outbound | Fund account creation + payout (amount, mode=NEFT/IMPS, purpose=payout) | API key + OAuth | 99.99% | Log failed payout with seller_id and reason; retry next Monday cycle; ops alert if > 5 failures per cycle |
| Razorpay Refunds API | REST API | Outbound | Refund creation (payment_id, amount, notes) | API key | 99.99% | Retry ×3; if all fail → mark refund as Manual Pending and alert ops |
| AWS S3 | AWS SDK (boto3 / aws-sdk-js) | Outbound | Image binary upload (multipart) | IAM Role (ECS task role, least privilege) | 99.99% | Reject product/store creation if S3 upload fails; no partial records created |
| AWS CloudFront | CDN | Delivery to buyer | Cached asset delivery | — (public distribution) | 99.9% | CloudFront cache miss falls back to S3 origin; no application-level fallback required |
| AWS SES | AWS SDK (boto3) | Outbound | Transactional email (HTML + plain text, <1MB per message) | IAM Role | 99.9% | Queue in memory; retry ×3 with 30s backoff; log failure with event_id after all retries |
| AWS ElastiCache Redis | redis-py / ioredis | Bidirectional | Cart data (JSON), session tokens, product catalog cache | VPC private subnet (no public access) | 99.9% | Graceful degradation: cart read-miss → reconstruct from DB; alert ops on consecutive cache failures |

---

## 7. Constraints (Formally Documented)

| ID | Constraint | Category | Source |
|----|-----------|----------|--------|
| CON-001 | Maximum AWS infrastructure spend: ₹1,66,000/month (~$2,000/month) at MVP scale (100 sellers) | Budget | CLAUDE.md |
| CON-002 | MVP delivery deadline: 2026-10-31 | Schedule | CLAUDE.md |
| CON-003 | Development team: 3 developers only (1 full-stack lead, 1 frontend, 1 backend); no dedicated DevOps or QA | Resource | CLAUDE.md |
| CON-004 | Geography: India only; no multi-currency, no international shipping | Scope | Phase 1 gap scan |
| CON-005 | Payment gateway: Razorpay exclusively; Stripe is not used | Technical | Phase 1 gap scan |
| CON-006 | Language: English only in MVP | Scope | Phase 2 Tier 3 inference |
| CON-007 | Architecture: Django monolith; microservices architecture is explicitly excluded | Technical | CLAUDE.md |
| CON-008 | PCI-DSS compliance delegated to Razorpay; ShopNest does not store card data | Compliance | Phase 1 feasibility |
| CON-009 | GST compliance: ShopNest must issue GST-compliant invoices for subscription billing before first paying seller | Legal | Indian GST Act |
| CON-010 | Consumer Protection (E-Commerce) Rules 2020 apply to ShopNest as a multi-vendor marketplace operator | Legal | Indian regulatory requirement |

---

*Sources: CLAUDE.md; Phase 1 ideation artifacts; Phase 2 gap scan answers (2026-05-04)*
*See `docs/assumptions/02-requirements-assumptions.md` for Tier 3 inference log.*
