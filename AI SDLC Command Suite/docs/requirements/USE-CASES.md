# Use Cases — ShopNest
**Phase:** 02 — Requirements Engineering
**Standard:** IEEE 29148:2018 · BABOK v3
**Generated:** 2026-05-04
**Status:** Draft — Awaiting Human Gate Approval

---

> Use cases document complex system interactions involving 3+ actors or 5+ steps. Each use case maps the primary flow and all significant alternative/error flows. BDD acceptance criteria in USER-STORIES.md cover individual feature testing; use cases cover end-to-end transaction integrity.

---

## UC-001 — Complete Guest Checkout Flow

| Attribute | Detail |
|-----------|--------|
| **Use Case ID** | UC-001 |
| **Name** | Complete Guest Checkout Flow |
| **Primary Actor** | Anjali (Guest Buyer) |
| **Secondary Actors** | Razorpay, AWS SES, Priya (Seller — notified), ShopNest Platform |
| **Business Goal** | Buyer completes a purchase without creating an account; order is created, payment confirmed, stock decremented, all parties notified |
| **User Stories Covered** | US-015, US-016, US-017, US-018, US-019 |
| **Business Rules Triggered** | BR-004, BR-008, BR-012, BR-018 |

**Preconditions:**
1. At least one Active product with stock > 0 exists in the marketplace
2. Razorpay sandbox/production account is active and payment API is reachable
3. AWS SES is configured and sending from a verified domain

**Main Flow (Happy Path):**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | Anjali | Opens /products/{slug} — product detail page | System renders SSR product page with price, images, Add to Cart button |
| 2 | Anjali | Clicks "Add to Cart" (no login) | System creates guest cart in Redis (TTL 24h); session cookie issued; cart total displayed |
| 3 | Anjali | Navigates to cart; reviews items; clicks "Checkout" | System validates stock for all cart items; all in stock — proceeds |
| 4 | Anjali | Enters checkout form: name, email, phone, shipping address | System validates all fields (phone = 10-digit, pincode = 6-digit) |
| 5 | ShopNest | Calls Razorpay Orders API with total amount in INR paise | Razorpay returns order_id; ShopNest stores pending order record |
| 6 | Anjali | Razorpay checkout modal opens; Anjali selects UPI and pays | Razorpay processes payment; sends payment.captured webhook to ShopNest |
| 7 | ShopNest | Validates Razorpay HMAC-SHA256 webhook signature | Signature valid — proceeds |
| 8 | ShopNest | Creates Order (Payment Confirmed) + sub-orders per seller; decrements stock atomically | DB transaction commits; stock updated; order IDs generated |
| 9 | ShopNest | Sends order confirmation email to Anjali via AWS SES | Email queued; delivered within 60s; contains order ID, items, tracking URL |
| 10 | ShopNest | Sends new-order alert email to each seller via AWS SES | Priya's new-order email delivered within 60s |
| 11 | ShopNest | Credits seller settlement ledger for each sub-order (pending seller confirmation) | Ledger entry created; amount = order amount net of Razorpay fee |

**Postconditions:**
- Order record exists in DB with status "Payment Confirmed"
- Product stock decremented correctly
- Anjali has a unique order tracking URL (in confirmation email)
- Priya sees the order in her seller dashboard
- Settlement ledger entry exists for Priya's sub-order

---

**Alternative Flow A — Stock Sold Out Between Cart Addition and Checkout (Step 3)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 3a | ShopNest | Validates stock at checkout initiation | Product X now has stock = 0 |
| 3b | System | Returns error: "Product X is no longer available" | Product X removed from Anjali's cart automatically |
| 3c | Anjali | Reviews updated cart; can proceed with remaining items or abandon | If cart is now empty: checkout blocked with "Cart is empty" message |

---

**Alternative Flow B — Payment Failure (Step 6)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 6a | Razorpay | Sends payment.failed webhook | ShopNest validates webhook signature |
| 6b | ShopNest | Marks pending order as "Payment Failed" | Stock quantities restored; cart preserved for retry |
| 6c | Anjali | Shown "Payment failed" message with retry button | Razorpay modal reopens for retry OR buyer abandons |

---

**Alternative Flow C — Razorpay API Unavailable (Step 5)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 5a | ShopNest | Razorpay Orders API returns error after 3 retries | ShopNest shows Anjali: "Payment service temporarily unavailable. Please try again." |
| 5b | ShopNest | Logs error with request details for ops review | Cart preserved; no order created |

---

**Alternative Flow D — Duplicate Webhook Delivery (Step 6–8)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 7d | Razorpay | Delivers payment.captured webhook a second time (retry) | ShopNest checks idempotency: order for this Razorpay payment_id already exists |
| 8d | ShopNest | Ignores duplicate; does NOT create second order | Returns HTTP 200 to Razorpay (acknowledge receipt); no further action |

---

## UC-002 — Seller Product Listing and Admin Approval

| Attribute | Detail |
|-----------|--------|
| **Use Case ID** | UC-002 |
| **Name** | Seller Product Listing and Admin Approval Workflow |
| **Primary Actor** | Priya (SME Seller) |
| **Secondary Actors** | Platform Admin, AWS SES |
| **Business Goal** | A seller submits a product listing; Platform Admin reviews and approves or rejects; approved products become visible to buyers |
| **User Stories Covered** | US-004, US-005, US-022, US-030 |
| **Business Rules Triggered** | BR-001, BR-015, BR-017 |

**Preconditions:**
1. Priya's seller account status = Active and subscription status = Active
2. Priya has completed the store setup wizard
3. Platform Admin is logged into the admin panel

**Main Flow — Product Submission and Approval:**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | Priya | Opens seller dashboard → Products → Add Product | System renders product creation form |
| 2 | Priya | Enters name, description, price, stock qty, category; uploads 2 images (JPEG ≤ 5MB each) | System validates MIME type and file size for each image |
| 3 | ShopNest | Uploads images to AWS S3; generates CloudFront URLs | Images stored; CloudFront URLs associated with product draft |
| 4 | Priya | Clicks "Submit for Review" | System creates product with status = "Pending Review"; product is NOT visible in marketplace |
| 5 | ShopNest | Adds product to Admin Product Approval Queue | Queue entry visible in admin panel within 5 seconds |
| 6 | Admin | Opens Product Approval Queue; reviews product name, images, description | Admin views product details with image thumbnails |
| 7 | Admin | Clicks "Approve" | System transitions product status → "Active" |
| 8 | ShopNest | Product appears in buyer-facing marketplace immediately | Category page and search index include the product |
| 9 | ShopNest | Sends "Product Approved" email to Priya via SES | Priya receives email within 60 seconds |

**Postconditions:**
- Product status = Active
- Product visible in marketplace category and search results
- Priya notified by email

---

**Alternative Flow A — Admin Rejects Product (Step 7)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 7a | Admin | Clicks "Reject" and enters reason: "Images are blurry. Please upload clearer photos." | System validates reason field (1–500 chars) |
| 7b | ShopNest | Transitions product status → "Rejected" | Product remains hidden from marketplace |
| 7c | ShopNest | Sends rejection email to Priya with the rejection reason | Priya receives email within 60 seconds |
| 7d | Priya | Edits product images and resubmits | Product transitions back to "Pending Review"; re-enters Admin queue |

---

**Alternative Flow B — Admin Removes Active Product (Post-Approval)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| B1 | Admin | Identifies policy-violating Active product; enters removal reason | System validates reason |
| B2 | ShopNest | Transitions product → "Admin Removed"; immediately hidden from marketplace | Buyer-facing pages no longer show product |
| B3 | ShopNest | Sends product removal email to Priya with reason | Priya notified within 60 seconds |

---

**Alternative Flow C — Price/Stock Edit (No Re-Review, BR-017)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| C1 | Priya | Edits price from ₹999 to ₹849 on an Active product | System validates new price (≥ ₹1) |
| C2 | ShopNest | Saves new price; product status remains "Active" | New price reflected on product page immediately; no admin notification |

---

## UC-003 — Seller Onboarding and Subscription

| Attribute | Detail |
|-----------|--------|
| **Use Case ID** | UC-003 |
| **Name** | Seller Onboarding: Registration → Store Setup → Subscription → Admin Approval |
| **Primary Actor** | Priya (SME Seller) |
| **Secondary Actors** | Razorpay Subscriptions API, Platform Admin, AWS SES |
| **Business Goal** | A new seller completes the full onboarding journey from account creation to first-time marketplace access |
| **User Stories Covered** | US-001, US-002, US-003, US-021 |
| **Business Rules Triggered** | BR-001, BR-007, BR-014 |

**Preconditions:**
1. Priya has not previously registered on ShopNest
2. Razorpay Subscriptions API is operational

**Main Flow:**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | Priya | Navigates to /register/seller; submits registration form | System validates unique email, password strength; creates seller account (status = Pending Admin Approval) |
| 2 | ShopNest | Adds Priya's application to Admin Seller Approval Queue | Queue entry visible to Admin |
| 3 | Admin | Reviews Priya's application; clicks Approve | System transitions Priya's seller status → Active |
| 4 | ShopNest | Sends "Account Approved" email to Priya | Priya receives email; prompted to log in and complete store setup |
| 5 | Priya | Logs in; is redirected to the Store Setup Wizard | Wizard checks: setup not completed → forces wizard before dashboard access |
| 6 | Priya | Completes wizard: store name, logo, banner, categories | System validates inputs; uploads images to S3 |
| 7 | Priya | Clicks "Continue to Subscription" | System calls Razorpay Subscriptions API: creates subscription plan + customer |
| 8 | Priya | Razorpay subscription checkout presented; Priya pays first month | Razorpay processes payment |
| 9 | Razorpay | Sends subscription.charged webhook to ShopNest | ShopNest validates HMAC signature |
| 10 | ShopNest | Sets Priya's subscription_status = Active | Priya can now access full seller dashboard |
| 11 | ShopNest | Generates GST subscription invoice; sends confirmation email | Priya receives invoice via email; downloadable from dashboard |
| 12 | Priya | Enters payout bank account details | System stores account number, IFSC, account holder name (encrypted at rest) |

**Postconditions:**
- Priya's seller account: status = Active, subscription = Active
- Store profile created with S3 images
- Razorpay Subscription object linked to Priya's account
- Bank details stored for settlement payouts
- Priya can list products (pending Admin approval per product)

---

## UC-004 — Order Fulfillment by Seller

| Attribute | Detail |
|-----------|--------|
| **Use Case ID** | UC-004 |
| **Name** | Order Fulfillment Workflow (Seller Side) |
| **Primary Actor** | Priya (SME Seller) |
| **Secondary Actors** | Anjali (Buyer), AWS SES |
| **Business Goal** | Seller receives a new order, confirms it, ships it, marks it delivered — buyer is informed at each step |
| **User Stories Covered** | US-007, US-008, US-009, US-033, US-034 |
| **Business Rules Triggered** | BR-002, BR-006 |

**Preconditions:**
1. An Order exists in "Payment Confirmed" status for Priya's store
2. Priya's subscription_status = Active

**Main Flow:**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | ShopNest | New order email delivered to Priya within 60s of payment | Priya opens email; sees order details |
| 2 | Priya | Opens seller dashboard → Orders; sees new order in "Payment Confirmed" | Dashboard shows order: buyer name, items, qty, total |
| 3 | Priya | Reviews order; clicks "Confirm Order" | System transitions sub-order → "Processing"; buyer can no longer cancel (BR-002) |
| 4 | ShopNest | Sub-order status = Processing; settlement ledger credit created for this sub-order | Ledger entry: amount = order value net of Razorpay fee |
| 5 | Priya | Packs order; ships via DTDC courier; enters AWB in dashboard: courier="DTDC", awb="DTC123456" | System validates courier name and AWB not empty |
| 6 | ShopNest | Transitions sub-order → "Shipped"; stores AWB and courier | Buyer tracking page now shows "Shipped — DTDC DTC123456" |
| 7 | ShopNest | Sends "Order Shipped" email to Anjali with AWB and courier name | Email delivered within 60 seconds |
| 8 | Priya | After delivery (buyer confirms via call/message); clicks "Mark Delivered" | System transitions sub-order → "Delivered" |

**Postconditions:**
- Sub-order status = Delivered
- AWB visible on buyer tracking page
- Settlement ledger entry credited for the delivered sub-order
- Sub-order included in next Monday payout cycle

---

**Alternative Flow A — Buyer Cancels Before Seller Confirmation (Step 3)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 3a | Anjali | Cancels order before Priya confirms | System validates: status = Payment Confirmed → cancellation allowed |
| 3b | ShopNest | Calls Razorpay Refunds API for full order amount | Refund initiated; order status = "Cancelled" |
| 3c | ShopNest | Restores stock quantities; removes sub-order from Priya's queue | Priya's dashboard no longer shows the order |
| 3d | ShopNest | Sends cancellation email to Anjali; removes settlement ledger entry | Anjali receives refund confirmation within 60 seconds |

---

## UC-005 — Weekly Seller Payout Settlement

| Attribute | Detail |
|-----------|--------|
| **Use Case ID** | UC-005 |
| **Name** | Weekly Seller Payout Settlement (Monday 09:00 IST) |
| **Primary Actor** | ShopNest Platform (automated scheduled job) |
| **Secondary Actors** | Razorpay Payouts API, Sellers (Priya), AWS SES |
| **Business Goal** | ShopNest disburses accumulated sales earnings to all eligible sellers' bank accounts on a weekly fixed cycle |
| **User Stories Covered** | US-024, US-032 |
| **Business Rules Triggered** | BR-005, BR-006, BR-013 |

**Preconditions:**
1. It is Monday 09:00 IST
2. At least one seller has a non-zero settlement balance with subscription_status = Active
3. Razorpay Payouts API is operational and ShopNest's Razorpay account has sufficient balance

**Main Flow:**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | ShopNest (cron) | Payout job triggers at Monday 09:00 IST | Job fetches all sellers with: settlement_balance > 0 AND subscription_status = Active |
| 2 | ShopNest | For each eligible seller: calls Razorpay Payouts API (mode: NEFT or IMPS, amount = settlement balance) | Razorpay returns payout_id; status = Initiated |
| 3 | ShopNest | Creates payout record: seller_id, amount, razorpay_payout_id, timestamp, status = Initiated | Payout record persisted to DB |
| 4 | ShopNest | Resets seller's settlement_balance to 0 for the disbursed amount | Ledger updated atomically |
| 5 | ShopNest | Sends "Payout Initiated" email to each seller with amount and payout reference ID | Emails delivered within 60 seconds of payout initiation |
| 6 | Seller | Receives bank credit within 1–2 business days (NEFT SLA) | Payout status updated to Completed via Razorpay webhook or polling |

**Postconditions:**
- Payout records in DB for all disbursements (status: Initiated → Completed)
- Sellers' settlement balances reset for disbursed amounts
- All sellers emailed with payout reference

---

**Alternative Flow A — Razorpay Payout API Fails for One Seller (Step 2)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 2a | Razorpay | Returns error for Seller X's payout (e.g., invalid IFSC) | ShopNest logs failure with seller_id, error reason, amount |
| 2b | ShopNest | Does NOT reset Seller X's settlement balance | Balance preserved for retry next cycle |
| 2c | ShopNest | Continues processing remaining sellers unaffected | Job completes for all other sellers |
| 2d | ShopNest | Sends ops alert: "Payout failed for seller {id}: {reason}" | Operations team investigates and contacts seller |

---

**Alternative Flow B — Suspended Seller Has Balance (BR-013)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1b | ShopNest | Job fetches eligible sellers; Priya is Suspended | Priya excluded from eligible list |
| 2b | ShopNest | Priya's balance is NOT disbursed | Balance preserved in ledger |
| 3b | Following Monday — Priya's subscription restored to Active | Priya is now eligible; full accumulated balance disbursed |

---

## UC-006 — Order Cancellation and Refund

| Attribute | Detail |
|-----------|--------|
| **Use Case ID** | UC-006 |
| **Name** | Buyer Order Cancellation and Razorpay Refund |
| **Primary Actor** | Anjali (Guest Buyer) |
| **Secondary Actors** | Razorpay Refunds API, ShopNest Platform, AWS SES |
| **Business Goal** | Buyer cancels an order before seller confirmation; full refund is processed via Razorpay; stock is restored |
| **User Stories Covered** | US-020 |
| **Business Rules Triggered** | BR-002, BR-003, BR-008 |

**Preconditions:**
1. Anjali has a confirmed order in "Payment Confirmed" status
2. The seller has NOT yet confirmed the order
3. Razorpay Refunds API is operational

**Main Flow:**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 1 | Anjali | Opens order tracking URL from confirmation email | System renders order status page: "Payment Confirmed — You can still cancel this order" |
| 2 | Anjali | Clicks "Cancel Order" button | System verifies order status = Payment Confirmed (cancellation permitted) |
| 3 | ShopNest | Calls Razorpay Refunds API: refund for full order amount, references original payment_id | Razorpay returns refund_id; refund queued |
| 4 | ShopNest | Transitions order status → "Cancelled" | Order visible as Cancelled on tracking page |
| 5 | ShopNest | Restores product stock quantities for all cancelled line items atomically | Stock incremented; products become purchasable again |
| 6 | ShopNest | Removes settlement ledger entries for this order's sub-orders | Sellers' pending balances adjusted accordingly |
| 7 | ShopNest | Sends cancellation + refund confirmation email to Anjali | Email: "Your order has been cancelled. Refund of ₹X initiated. Expected credit: 5–7 business days." |

**Postconditions:**
- Order status = Cancelled
- Razorpay refund initiated for full amount
- Stock quantities restored
- Settlement ledger updated (sub-order credit removed)
- Anjali notified by email

---

**Alternative Flow A — Cancellation Attempted After Seller Confirmation**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 2a | Anjali | Attempts to cancel order already in "Processing" status | System returns error: "This order cannot be cancelled — your seller is already processing it." |
| 2b | System | No refund initiated; no status change | Order remains Processing |

---

**Alternative Flow B — Razorpay Refund API Fails (Step 3)**

| Step | Actor | Action | System Response |
|------|-------|--------|-----------------|
| 3a | Razorpay | Returns error after 3 retry attempts | ShopNest marks refund as "Manual Pending" |
| 3b | ShopNest | Still transitions order to Cancelled; still restores stock | Stock and order state updated correctly |
| 3c | ShopNest | Sends ops alert: "Manual refund required for order {id}, amount ₹{X}" | Operations team manually processes refund via Razorpay dashboard |
| 3d | ShopNest | Sends email to Anjali acknowledging cancellation, noting refund processing delay | "Cancellation confirmed. Refund is being processed — you will receive it within 5–10 business days." |

---

*Sources: Phase 1 ideation artifacts; Phase 2 gap scan (2026-05-04); REQUIREMENTS.md functional requirements*  
*All use cases verified against BUSINESS-RULES.md authority catalog.*
