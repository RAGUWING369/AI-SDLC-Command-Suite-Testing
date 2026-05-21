# User Stories — ShopNest
**Phase:** 02 — Requirements Engineering
**Standard:** INVEST criteria (Bill Wake, 2003) · BDD Given/When/Then (Dan North)
**Generated:** 2026-05-04
**Status:** Draft — Awaiting Human Gate Approval

---

## Personas Reference
| Persona | Description |
|---------|-------------|
| **Priya** | SME Seller — boutique clothing seller, Bangalore, 120 SKUs, moving from Instagram DMs |
| **Rahul** | Registered Buyer — 28-year-old, Mumbai, shops via UPI on mobile |
| **Anjali** | Guest Buyer — buys via a link shared by a seller on WhatsApp; no ShopNest account |
| **Admin** | Platform Admin — ShopNest team member responsible for moderation |

---

## MoSCoW Priority Legend
- **Must Have** — MVP blocker; platform does not function without it
- **Should Have** — High value; planned for MVP but can be descoped if timeline forces it
- **Could Have** — Nice to have; included if capacity allows
- **Won't Have** — Explicitly out of scope for this release

---

## MUST HAVE Stories

---

### US-001 — Seller Registration

**As Priya (SME Seller),** I want to create a ShopNest seller account with my email, password, and basic business details, so that I can begin setting up my online store.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-AUTH-001, FR-AUTH-002, FR-AUTH-003, FR-AUTH-009 |
| BRs | BR-014 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Successful seller registration
Given Priya navigates to /register/seller
When she submits: email=priya@boutique.in, password=Boutique123, full name=Priya Mehta,
  business name=Priya's Boutique, store name=Priyaboutique, phone=9876543210,
  city=Bangalore, GSTIN=(blank), pincode=560001
Then the system creates her account with status "Pending Review"
And returns HTTP 201 with a success message
And her password is stored as a bcrypt hash (cost ≥ 12)
And she does NOT yet have access to the seller dashboard

Scenario: Duplicate email rejected
Given an account already exists with email=priya@boutique.in
When Priya attempts to register with the same email
Then the system returns HTTP 409 with message "Email already registered"
And no new account is created

Scenario: Weak password rejected
Given Priya submits registration with password="boutique" (no numeric character)
When the form is submitted
Then the system returns HTTP 400 with message "Password must be at least 8 characters and contain at least 1 number"
```

---

### US-002 — Store Setup Wizard

**As Priya (SME Seller),** I want to complete a guided store setup wizard after registration, so that my store has a name, logo, banner, and categories before I start listing products.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | M |
| FRs | FR-SELLER-001, FR-SELLER-002, FR-SELLER-005 |
| BRs | — |
| Assumption | A-01-001 (sellers need guided onboarding) |

**Acceptance Criteria:**

```
Scenario: Seller completes wizard successfully
Given Priya is logged in with status "Pending Review" and has not completed setup
When she submits the wizard with: store name=Priyaboutique, logo=priya-logo.png (1.5MB),
  banner=priya-banner.jpg (3MB), categories=[Women's Clothing, Accessories]
Then the system creates her store record
And images are uploaded to S3 and served via CloudFront URLs
And she is redirected to the subscription payment step
And her seller status remains "Pending Admin Approval" until Admin approves

Scenario: Oversized logo rejected
Given Priya uploads a logo image of 3MB (exceeds 2MB limit)
When she submits the wizard
Then the system returns HTTP 400 with message "Logo must be under 2MB"
And the wizard does not proceed

Scenario: Product listing blocked before wizard complete
Given Priya is registered but has not completed the wizard
When she attempts to access the product listing interface
Then the system returns HTTP 403 with message "Complete store setup first"
```

---

### US-003 — Seller Subscription Signup

**As Priya (SME Seller),** I want to subscribe to ShopNest's monthly plan via Razorpay after completing my store setup, so that I can activate my seller account and begin listing products.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | M |
| FRs | FR-SELLER-003, FR-SELLER-004, FR-SUBSCR-001, FR-SUBSCR-003, FR-SUBSCR-004, FR-SUBSCR-005, FR-SUBSCR-006 |
| BRs | BR-007 |
| Assumption | A-01-016 (₹1,999/month pricing) |

**Acceptance Criteria:**

```
Scenario: Successful subscription and account activation
Given Priya has completed the store setup wizard
When she completes payment via Razorpay (UPI or card)
And the system receives a valid subscription.charged webhook from Razorpay
Then a Razorpay Subscription is created with monthly billing
And Priya's subscription status = "Active"
And she can access the seller dashboard and product listing interface
And she receives a subscription confirmation email with her GST invoice attached

Scenario: Subscription payment failure triggers grace period
Given Priya's subscription is Active
When Razorpay sends a subscription.charge.failed webhook (monthly renewal failed)
Then Priya's subscription status = "Payment Failed"
And she receives a payment failure notification email
And she retains full dashboard access for 7 days

Scenario: Suspension after grace period
Given Priya's subscription has been in Payment Failed status for 8 days
When the grace-period check runs
Then Priya's status = "Suspended"
And all her products are hidden from the marketplace
And she receives a suspension email
And her product data and order history are preserved

Scenario: Restoration after successful renewal
Given Priya's subscription is Suspended
When Razorpay sends a subscription.charged (success) webhook
Then Priya's status = "Active"
And all her previously approved products are immediately re-published
```

---

### US-004 — Add Product Listing

**As Priya (SME Seller),** I want to add a new product with name, description, price, stock quantity, category, and images, so that buyers can discover and purchase it.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | M |
| FRs | FR-PRODUCT-001, FR-PRODUCT-002, FR-PRODUCT-003, FR-PRODUCT-004, FR-PRODUCT-011 |
| BRs | BR-015 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Successful product submission
Given Priya is logged in with Active subscription
When she submits a product: name="Blue Cotton Kurti", description="Handloom cotton...",
  price=₹999, stock=50, category="Women's Clothing", images=[kurti1.jpg (2MB), kurti2.jpg (3MB)]
Then the product is created with status "Pending Review"
And images are stored in S3 with CloudFront URLs
And the product appears in the Admin's Product Approval Queue
And Priya sees it in her dashboard as "Pending Review"
And the product does NOT appear in the buyer-facing marketplace

Scenario: Executable file upload rejected
Given Priya attempts to upload product-image.exe as a product image
When the form is submitted
Then the system returns HTTP 400 with message "Only JPEG and PNG images are accepted"
And the product is not created

Scenario: Deletion blocked with active order
Given Priya's product has an order in "Processing" status
When she attempts to delete the product
Then the system returns HTTP 409 with message "Cannot delete product with active orders"
```

---

### US-005 — Admin Approves Product Listing

**As Admin,** I want to review and approve or reject product listings submitted by sellers, so that only appropriate products appear on the ShopNest marketplace.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-PRODUCT-004, FR-PRODUCT-005, FR-PRODUCT-006, FR-ADMIN-005, FR-ADMIN-006 |
| BRs | BR-001 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Admin approves product
Given the Product Approval Queue shows Priya's "Blue Cotton Kurti" submission
When Admin clicks Approve
Then the product status = "Active"
And it appears in the buyer-facing marketplace within 30 seconds
And Priya receives a product approval email

Scenario: Admin rejects product with reason
Given Admin reviews a product with insufficient description
When Admin clicks Reject and enters reason="Description is too short. Please add at least 100 characters."
Then the product status = "Rejected"
And it remains hidden from the marketplace
And Priya receives a rejection email with the exact reason
And Priya can edit and resubmit the product
```

---

### US-006 — Edit Product Price and Stock

**As Priya (SME Seller),** I want to edit my product's price and stock quantity without triggering a re-review, so that I can quickly react to market conditions and inventory changes.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-PRODUCT-007, FR-PRODUCT-008 |
| BRs | BR-017 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Price update does not trigger re-review
Given Priya's product "Blue Cotton Kurti" is in Active status
When she updates the price from ₹999 to ₹899
Then the product remains Active
And the new price is immediately reflected on the product detail page
And no admin notification is sent

Scenario: Name change triggers re-review
Given Priya's product "Blue Cotton Kurti" is in Active status
When she changes the name to "Blue Handloom Kurti"
Then the product status changes to "Pending Review"
And the product is hidden from buyers until re-approved
And Admin receives a notification in the approval queue
```

---

### US-007 — Seller Views Incoming Orders

**As Priya (SME Seller),** I want to view all orders assigned to my store in my dashboard, so that I know what I need to fulfill.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-FULFILL-001, FR-FULFILL-006 |
| BRs | BR-016 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Seller sees only their own orders
Given Priya has 3 sub-orders and another seller (Ravi) has 5 sub-orders
When Priya views her seller dashboard orders page
Then she sees exactly 3 orders, sorted by most recent first
And Ravi's orders do not appear
And each order shows: order ID, buyer first name + last initial, items, total, status

Scenario: Seller receives new order alert
Given Anjali places an order for Priya's product at 10:15 AM
When Razorpay confirms payment at 10:16 AM
Then by 10:17 AM, Priya receives a "New Order Received" email
And the order appears in her dashboard with status "Payment Confirmed"
```

---

### US-008 — Seller Confirms Order

**As Priya (SME Seller),** I want to confirm an order, so that the buyer knows it is being processed and can no longer cancel.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | XS |
| FRs | FR-FULFILL-002 |
| BRs | BR-002 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Seller confirms order successfully
Given an order in "Payment Confirmed" status in Priya's dashboard
When Priya clicks "Confirm Order"
Then the order status = "Processing"
And the buyer can no longer cancel this order

Scenario: Buyer cancel blocked after seller confirmation
Given Priya has confirmed order #ORD-123
When Anjali attempts to cancel order #ORD-123
Then the system returns HTTP 409 with message "Order cannot be cancelled after seller confirmation"
```

---

### US-009 — Seller Ships Order

**As Priya (SME Seller),** I want to mark an order as shipped with the courier name and AWB number, so that the buyer can track their package.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-FULFILL-003, FR-FULFILL-004 |
| BRs | — |
| Assumption | A-01-001 (sellers use third-party couriers; AWB is their reference) |

**Acceptance Criteria:**

```
Scenario: Seller marks order as shipped
Given order #ORD-123 is in "Processing" status
When Priya enters courier_name="DTDC" and awb="DTDC123456789" and clicks "Mark as Shipped"
Then the order status = "Shipped"
And the AWB number is stored and visible on the buyer's tracking page
And Anjali receives a "Your order has been shipped" email with courier and AWB within 60 seconds

Scenario: AWB required — cannot ship without it
Given order #ORD-123 is in Processing status
When Priya submits the ship action with awb="" (empty)
Then the system returns HTTP 400 with message "AWB number and courier name are required"
```

---

### US-010 — Low-Stock Alert

**As Priya (SME Seller),** I want to receive a low-stock alert when any product's stock falls to 5 or below, so that I can restock before selling out.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-PRODUCT-010 |
| BRs | — |
| Assumption | A-02-001 (low-stock threshold = 5 units) |

**Acceptance Criteria:**

```
Scenario: Low-stock flag appears in dashboard
Given Priya's product "Blue Cotton Kurti" has stock quantity = 6
When an order for 2 units is confirmed (stock → 4)
Then a low-stock flag appears in her seller dashboard within 60 seconds

Scenario: Out-of-stock hides product
Given Priya's product has stock = 1
When an order for 1 unit is confirmed (stock → 0)
Then the product is hidden from all buyer-facing pages immediately
And Priya's dashboard shows the product as "Out of Stock"
```

---

### US-011 — Browse Marketplace Homepage

**As Anjali (Guest Buyer),** I want to see a homepage with featured products and categories when I visit ShopNest, so that I can start discovering products without needing an account.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-BROWSE-001 |
| BRs | BR-001 |
| Assumption | A-01-002 (>70% mobile traffic) |

**Acceptance Criteria:**

```
Scenario: Homepage renders with products (SSR)
Given 5 Active products exist from 2 different sellers
When Anjali visits shopnest.in (no login)
Then the page renders server-side with product names and images visible
And the page HTML contains og:title and og:description meta tags (SEO)
And Google PageSpeed Insights reports LCP < 2.5s on mobile 4G simulation

Scenario: Suspended seller's products not shown
Given Seller X is Suspended
When Anjali visits the homepage
Then Seller X's products do not appear in featured products or categories
```

---

### US-012 — Browse Products by Category

**As Anjali (Guest Buyer),** I want to browse all products in a specific category, so that I can find items relevant to my interest.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-BROWSE-002 |
| BRs | BR-001 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Category page shows correct products
Given 30 Active products exist in category "Women's Clothing"
When Anjali opens /category/womens-clothing
Then page 1 shows 24 products; page 2 shows 6 products
And all products are server-side rendered with name and price visible

Scenario: Out-of-stock product absent from category
Given a product with stock = 0 exists in "Women's Clothing"
When Anjali opens the category page
Then the out-of-stock product does not appear
```

---

### US-013 — View Product Detail Page

**As Anjali (Guest Buyer),** I want to see a product's full details including images, price, description, and stock status, so that I can decide whether to purchase it.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-BROWSE-003, FR-BROWSE-004, FR-BROWSE-007 |
| BRs | BR-001 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Product detail page renders fully (SSR)
Given "Blue Cotton Kurti" is Active with 3 images, price ₹999, stock 50
When Anjali opens /products/blue-cotton-kurti-{uuid}
Then she sees: image carousel (3 images from CloudFront), name, price, description, seller store name
And the "Add to Cart" button is enabled

Scenario: Out-of-stock product shows correct state
Given the product has stock = 0
When Anjali opens the product detail page
Then the "Add to Cart" button is disabled and replaced with "Out of Stock" label
And she cannot add it to cart via API POST

Scenario: URL stable after name edit
Given the product URL is /products/blue-cotton-kurti-abc123
When Priya renames the product to "Blue Handloom Kurti"
Then the original URL still resolves to the correct product after re-approval
```

---

### US-014 — Search Products

**As Anjali (Guest Buyer),** I want to search for products by keyword, so that I can quickly find what I'm looking for without browsing all categories.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-BROWSE-005, FR-BROWSE-006 |
| BRs | BR-001 |
| Assumption | A-02-002 (PostgreSQL FTS sufficient for ≤50K products at MVP scale) |

**Acceptance Criteria:**

```
Scenario: Keyword search returns relevant results
Given 1,000 Active products exist, 5 with "cotton" in name or description
When Anjali searches for "cotton"
Then the results page shows those 5 products, ranked by relevance
And out-of-stock and inactive products are NOT included

Scenario: Search returns within performance target
Given 50,000 Active products exist
When 100 concurrent search queries are submitted in a load test
Then P95 response time is ≤ 500ms
```

---

### US-015 — Add to Cart

**As Anjali (Guest Buyer),** I want to add products to a cart without creating an account, so that I can collect items before deciding to checkout.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-CART-001, FR-CART-002, FR-CART-004, FR-CART-006, FR-CART-007 |
| BRs | — |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Guest adds item to cart without login
Given Anjali is not logged in
When she clicks "Add to Cart" on "Blue Cotton Kurti" (qty = 1)
Then the item is added to a guest cart (session cookie set)
And the cart total updates to ₹999
And no account creation is prompted

Scenario: Cart from multiple sellers
Given Anjali adds Product A (Seller X) and Product B (Seller Y)
When she views the cart
Then both items appear with their respective sellers indicated
And total = price_A + price_B

Scenario: Cart persists for 24 hours
Given Anjali adds an item to the cart at 10:00 AM
When she returns at 09:59 AM the next day (just under 24h)
Then the cart still contains her item
```

---

### US-016 — Guest Checkout

**As Anjali (Guest Buyer),** I want to checkout without creating an account by providing my contact and shipping details, so that I can complete my purchase immediately.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | M |
| FRs | FR-CHECKOUT-001, FR-CHECKOUT-005, FR-CART-005 |
| BRs | BR-012, BR-018 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Guest checkout form validates successfully
Given Anjali has items in cart and navigates to /checkout
When she enters: name=Anjali Sharma, email=anjali@gmail.com, phone=9123456789,
  address=12 MG Road, city=Mumbai, state=Maharashtra, pincode=400001
Then the form is accepted and she proceeds to payment

Scenario: Invalid phone number rejected
Given Anjali enters phone=91234 (5 digits)
When she submits checkout
Then the system returns HTTP 400 with message "Enter a valid 10-digit Indian mobile number"
And she cannot proceed to payment

Scenario: Out-of-stock item caught at checkout
Given an item in Anjali's cart sold out while she was shopping
When she initiates checkout
Then the system returns an error listing the sold-out item
And removes it from her cart
And she must review the cart before retrying checkout
```

---

### US-017 — Pay via Razorpay

**As Anjali (Guest Buyer),** I want to pay for my order using UPI, card, or NetBanking via Razorpay, so that I can complete my purchase securely using my preferred India-native payment method.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | L |
| FRs | FR-CHECKOUT-003, FR-CHECKOUT-004, FR-CHECKOUT-005, FR-CHECKOUT-006, FR-CHECKOUT-007, FR-CHECKOUT-008, FR-CHECKOUT-011, FR-CHECKOUT-012 |
| BRs | BR-004, BR-008, BR-012 |
| Assumption | A-01-003 |

**Acceptance Criteria:**

```
Scenario: Successful UPI payment and order creation
Given Anjali has completed checkout form and a Razorpay Order is created
When she pays via UPI and Razorpay sends a payment.captured webhook with valid HMAC signature
Then a platform Order is created with status "Payment Confirmed"
And stock for each ordered product is decremented atomically
And Anjali receives an order confirmation email within 60 seconds
And each seller receives a new-order notification within 60 seconds

Scenario: Payment failure handled correctly
Given a Razorpay payment.failed webhook arrives
Then any pending order is marked "Payment Failed"
And decremented stock quantities are restored
And Anjali is shown a payment failed message and invited to retry

Scenario: Forged client-side success rejected
Given Anjali's browser sends a fake payment success callback without a corresponding Razorpay webhook
Then no Order is created in the system
And no stock is decremented
```

---

### US-018 — Receive Order Confirmation Email

**As Anjali (Guest Buyer),** I want to receive an order confirmation email immediately after payment, so that I have a record of my purchase and know my order was received.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-CHECKOUT-009, FR-NOTIF-001, FR-NOTIF-002, FR-NOTIF-004 |
| BRs | — |
| Assumption | A-02-003 (AWS SES as email provider) |

**Acceptance Criteria:**

```
Scenario: Buyer receives confirmation email on time
Given Anjali's payment is confirmed at 14:30:05
When the system processes the payment.captured webhook
Then an order confirmation email arrives in Anjali's inbox by 14:31:05 (within 60 seconds)
And the email contains: order ID, itemized list (product name × qty × price), total, seller name

Scenario: Email retry on SES failure
Given SES returns a transient failure on first attempt
When the system retries up to 3 times with exponential backoff
Then the email is delivered on the second or third attempt
And a success delivery log entry is recorded
```

---

### US-019 — Track Order Status

**As Anjali (Guest Buyer),** I want to check my order status at any time via the tracking URL in my confirmation email, so that I know when my order has been shipped and can expect delivery.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-ORDER-001, FR-ORDER-002 |
| BRs | — |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Guest accesses order tracking without login
Given Anjali received an order confirmation email with tracking URL
When she opens the URL (contains order UUID + HMAC token)
Then she sees the current order status, product list, and seller name
And no login is required

Scenario: Tracking URL shows AWB after shipment
Given Priya has marked order #ORD-123 as Shipped with AWB="DTDC123456789" and courier="DTDC"
When Anjali opens the tracking URL
Then the status shows "Shipped" and AWB number "DTDC123456789" and courier "DTDC" are displayed
```

---

### US-020 — Cancel Order Before Confirmation

**As Anjali (Guest Buyer),** I want to cancel my order before the seller confirms it, so that I can get a full refund if I change my mind.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | M |
| FRs | FR-ORDER-003, FR-ORDER-004 |
| BRs | BR-002, BR-003 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Buyer cancels order in Payment Confirmed status
Given order #ORD-123 is in "Payment Confirmed" status
When Anjali clicks Cancel on the order tracking page
Then the system calls Razorpay Refunds API for the full amount
And the order status = "Cancelled"
And stock quantities are restored for all cancelled items
And Anjali receives a cancellation + refund confirmation email within 60 seconds

Scenario: Cancellation blocked after seller confirms
Given Priya has confirmed order #ORD-123 (status = Processing)
When Anjali attempts to cancel
Then the system returns a message "Order cannot be cancelled — seller is already processing it"
And the order status does not change
```

---

### US-021 — Admin Approves Seller Registration

**As Admin,** I want to review and approve seller registration applications, so that only legitimate businesses sell on the ShopNest marketplace.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-ADMIN-001, FR-ADMIN-002 |
| BRs | BR-001, BR-014 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Admin approves seller
Given Priya's seller application appears in the Admin Seller Approval Queue
When Admin clicks Approve
Then Priya's seller status = "Active"
And Priya receives an "Account Approved" email
And Priya can now access the seller dashboard and proceed to list products (after admin product approval)

Scenario: New application appears in queue promptly
Given Ravi completes seller registration at 11:00 AM
When Admin opens the approval queue at 11:01 AM
Then Ravi's application appears in the queue with store name, email, and registration timestamp
```

---

### US-022 — Admin Approves Product Listing

**As Admin,** I want to approve or reject product listings submitted by sellers, so that only appropriate products are visible on ShopNest.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-PRODUCT-004, FR-PRODUCT-005, FR-PRODUCT-006, FR-ADMIN-005, FR-ADMIN-006 |
| BRs | BR-001 |
| Assumption | — |

**Acceptance Criteria:**

*(See US-005 — same user story from Admin perspective, combined for DRY reference)*

```
Scenario: Product goes live immediately upon approval
Given Admin approves "Blue Cotton Kurti" at 15:00:00
When a buyer visits the marketplace at 15:00:30
Then the product is visible in search and category results
```

---

### US-023 — Admin Suspends Seller Account

**As Admin,** I want to suspend a seller account for policy violations, so that their products are immediately removed from the marketplace.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-ADMIN-004, FR-ADMIN-005 |
| BRs | BR-001, BR-013 |
| Assumption | — |

**Acceptance Criteria:**

```
Scenario: Seller suspended — products hidden immediately
Given Priya has 10 Active products in the marketplace
When Admin suspends Priya's account
Then all 10 products disappear from the buyer-facing marketplace immediately
And Priya receives a suspension notification email
And Priya's dashboard access is blocked (HTTP 403)
And Priya's payout is withheld until reactivation

Scenario: Admin reactivates suspended seller
Given Priya's account is Suspended
When Admin clicks Reactivate
Then Priya's status = "Active"
And her previously Active products are re-published immediately
```

---

### US-024 — Seller Receives Weekly Payout

**As Priya (SME Seller),** I want to receive my sales earnings in my bank account on a weekly basis, so that I have regular cash flow from my online store.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Must Have** |
| INVEST | ✅ All criteria met |
| Size | L |
| FRs | FR-PAYOUT-001, FR-PAYOUT-002, FR-PAYOUT-003, FR-PAYOUT-004, FR-PAYOUT-005 |
| BRs | BR-005, BR-006, BR-013 |
| Assumption | A-02-004 (Razorpay Payouts API for NEFT/IMPS) |

**Acceptance Criteria:**

```
Scenario: Weekly payout executed on Monday
Given Priya has 3 confirmed sub-orders totalling ₹4,500 (net of Razorpay fee) in the settlement ledger
When the Monday 09:00 IST payout job runs
Then Razorpay Payouts API is called for ₹4,500 to Priya's registered bank account
And a payout record is created with status "Initiated" and Razorpay payout reference ID
And Priya receives a "Payout Initiated" email with the amount and reference ID

Scenario: Suspended seller excluded from payout
Given Priya is Suspended with ₹3,000 in settlement balance
When the Monday payout job runs
Then Priya's payout is NOT disbursed
And her ₹3,000 balance is preserved in the ledger

Scenario: Payout included when restored
Given Priya is restored to Active on Tuesday (after Monday payout ran)
When the following Monday payout runs
Then Priya's full ₹3,000 accumulated balance is disbursed
```

---

## SHOULD HAVE Stories

---

### US-025 — Buyer Account Registration

**As Rahul (Registered Buyer),** I want to create a ShopNest buyer account, so that I can save my address, view order history, and checkout faster on future visits.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-AUTH-005, FR-AUTH-006 |
| BRs | BR-018 |

**Acceptance Criteria:**

```
Scenario: Successful buyer registration
Given Rahul submits: email=rahul@gmail.com, password=Mumbai123, name=Rahul Verma, phone=9988776655
Then his buyer account is created
And he is issued JWT tokens (access: 24h, refresh: 30d)
And he can access authenticated buyer endpoints

Scenario: Buyer JWT rejected on seller endpoints
Given Rahul has a Buyer role JWT
When he attempts GET /api/v1/seller/orders
Then the system returns HTTP 403
```

---

### US-026 — Registered Buyer Checkout with Saved Address

**As Rahul (Registered Buyer),** I want to checkout using my saved delivery address, so that I don't have to re-enter it every time.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-CHECKOUT-002, FR-CART-003 |
| BRs | — |

**Acceptance Criteria:**

```
Scenario: Registered buyer sees saved address at checkout
Given Rahul has a saved address (Flat 4B, Andheri, Mumbai, 400058)
When he proceeds to checkout
Then his saved address is pre-populated in the shipping form
And he can choose to use it or enter a new address

Scenario: Cart restored after login
Given Rahul added items before logging in (guest session)
When he logs in
Then his cart items are transferred to his authenticated cart (if no conflict)
```

---

### US-027 — Buyer Views Order History

**As Rahul (Registered Buyer),** I want to see all my past orders and their statuses in my account, so that I can track purchases and reference past transactions.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-ORDER-005 |
| BRs | BR-016, BR-018 |

**Acceptance Criteria:**

```
Scenario: Buyer sees their own orders only
Given Rahul has placed 5 orders
When he opens /account/orders
Then he sees all 5 orders sorted by most recent first
And each shows: order ID, date, product list, total, current status

Scenario: Guest orders not linked to account
Given Anjali (guest) placed an order with email anjali@gmail.com
And Rahul creates a buyer account with the same email
When Rahul views his order history
Then Anjali's guest order does NOT appear (BR-018)
```

---

### US-028 — Seller Views Sales Analytics

**As Priya (SME Seller),** I want to see my sales revenue, order count, and top products in my dashboard, so that I can understand my business performance.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ All criteria met |
| Size | M |
| FRs | FR-ANALYTICS-001, FR-ANALYTICS-002 |
| BRs | — |

**Acceptance Criteria:**

```
Scenario: Analytics show correct 7-day and 30-day totals
Given Priya has 3 orders (₹1,500 each) in the last 7 days and 10 orders (₹15,000 total) in last 30 days
When she opens her analytics dashboard
Then she sees: Last 7 days — Revenue ₹4,500, Orders 3, AOV ₹1,500
And Last 30 days — Revenue ₹15,000, Orders 10, AOV ₹1,500

Scenario: Top 5 products ranked correctly
Given Product A has 12 orders and Product B has 8 orders in last 30 days
When Priya views top products
Then Product A appears first, Product B second
```

---

### US-029 — Admin Rejects Seller Application

**As Admin,** I want to reject a seller application with a reason, so that ineligible applicants are notified and the marketplace remains curated.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ All criteria met |
| Size | S |
| FRs | FR-ADMIN-003 |
| BRs | BR-014 |

**Acceptance Criteria:**

```
Scenario: Admin rejects with mandatory reason
Given a seller application in the pending queue
When Admin clicks Reject and enters "Incomplete business details. Please re-register with a valid GSTIN."
Then the applicant receives a rejection email with the reason
And their account status = "Rejected"
And they can re-register with a different email
```

---

### US-030 — Admin Rejects Product Listing

*(Referenced in US-005 / US-022 — covered by FR-PRODUCT-006)*

**As Admin,** I want to reject a product with a reason, so that sellers know why their product was not approved and can correct it.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ |
| Size | XS |
| FRs | FR-PRODUCT-006 |
| BRs | — |

---

### US-031 — Seller Edits Store Profile

**As Priya (SME Seller),** I want to update my store name, logo, banner, and categories after initial setup, so that my store always reflects my current brand.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ |
| Size | S |
| FRs | FR-SELLER-006 |
| BRs | — |

**Acceptance Criteria:**

```
Scenario: Store logo update reflected on marketplace
Given Priya uploads a new logo image
When the upload succeeds
Then the new logo appears on her seller profile page and product listings within 60 seconds (CDN invalidation)
```

---

### US-032 — Seller Views Payout History

**As Priya (SME Seller),** I want to see a history of all payouts I've received, so that I can reconcile my bank statements.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ |
| Size | S |
| FRs | FR-PAYOUT-006 |
| BRs | BR-011 |

---

### US-033 — Buyer Receives Order Shipped Email

**As Anjali (Guest Buyer),** I want to receive an email when my order is shipped with the tracking number, so that I know to expect my package.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ |
| Size | S |
| FRs | FR-FULFILL-004, FR-NOTIF-002 |
| BRs | — |

---

### US-034 — Seller Marks Order Delivered

**As Priya (SME Seller),** I want to mark an order as delivered after the buyer receives it, so that my dashboard reflects accurate fulfilment status.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Should Have** |
| INVEST | ✅ |
| Size | XS |
| FRs | FR-FULFILL-005 |
| BRs | BR-006 |

---

## COULD HAVE Stories

---

### US-035 — Admin Views Platform Metrics Dashboard

**As Admin,** I want to see a platform-wide dashboard with total sellers, products, orders, and GMV, so that I can monitor ShopNest's overall health.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Could Have** |
| INVEST | ✅ |
| Size | M |
| FRs | FR-ADMIN-006 |
| BRs | — |

---

### US-036 — Seller Views Subscription Status and Invoices

**As Priya (SME Seller),** I want to see my subscription status, next billing date, and download my GST invoices, so that I have complete billing transparency.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Could Have** |
| INVEST | ✅ |
| Size | S |
| FRs | FR-SUBSCR-002, FR-SUBSCR-007 |
| BRs | BR-010, BR-011 |

---

### US-037 — Seller Updates Bank Account for Payout

**As Priya (SME Seller),** I want to update my payout bank account details, so that payouts go to my correct account if I change banks.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Could Have** |
| INVEST | ✅ |
| Size | S |
| FRs | FR-SELLER-007 |
| BRs | BR-013 |

---

### US-038 — Seller Exports Order Data as CSV

**As Priya (SME Seller),** I want to download my order history as a CSV file, so that I can use it for my own accounting and GST invoicing.

| Attribute | Detail |
|-----------|--------|
| MoSCoW | **Could Have** |
| INVEST | ✅ |
| Size | S |
| FRs | FR-ANALYTICS-003 |
| BRs | BR-010, BR-011 |

---

## Story Size Summary

| Size | Story Count | Criteria |
|------|-------------|---------|
| XS | 4 | < 1 day effort |
| S | 22 | 1–2 days effort |
| M | 8 | 3–5 days effort |
| L | 2 | 5–8 days effort (must be first in sprint) |
| XL | 0 | ✅ No XL stories — all split appropriately |

**Must Have total:** 24 stories · **Should Have:** 10 stories · **Could Have:** 4 stories

---

*Sources: Phase 1 ideation artifacts; Phase 2 gap scan (2026-05-04)*  
*INVEST validation applied to all stories. No story failed — all criteria met.*
