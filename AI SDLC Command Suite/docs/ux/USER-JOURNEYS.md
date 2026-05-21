# USER-JOURNEYS.md — ShopNest UX Design

**Phase:** 05 — UX Design (Full Production Run)
**Date:** 2026-05-14
**Screens Covered:** All 21 screens (SCR-001 through SCR-021)
**Personas Covered:** Rahul (Registered Buyer) · Anjali (Guest Buyer) · Priya (SME Seller) · Admin

---

## Persona Reference

| Persona | Role | Key Needs | Device | Context |
|---------|------|-----------|--------|---------|
| **Rahul** | Registered Buyer, 28, Mumbai | Fast mobile checkout via UPI, clear product info, reliable order tracking | Mobile (375px), 4G | Shops during commute or lunch breaks |
| **Anjali** | Guest Buyer, clicks WhatsApp link from seller | Buy without creating account, minimal friction, trust signals | Mobile (375px), 3G/4G | One-time or occasional purchase via seller share |
| **Priya** | SME Seller, Bangalore, 120 SKUs | Register quickly, list products, track orders, see payment status, no technical complexity | Mobile + Desktop | Manages store between other tasks |
| **Admin** | ShopNest Team, internal moderator | Review and approve/reject sellers and products efficiently, monitor platform health | Desktop | Works during business hours, processes queue daily |

---

## Journey 1: Rahul Discovers and Purchases a Product

**Entry Point:** Homepage or category page → Product Detail  
**Exit Point:** Order confirmation  
**Screens touched:** SCR-004 (Homepage) → SCR-005 (Category) → SCR-001 (PDP) → SCR-002 (Cart → Checkout → Success)

### Happy Path

| Step | Action | Screen | Emotional State | System Response |
|------|--------|--------|----------------|----------------|
| 1 | Opens ShopNest app / homepage | SCR-004 | Curious, browsing | SSR homepage renders featured products and categories in < 2.5s |
| 2 | Taps "Women's Clothing" category banner | SCR-005 | Exploring | Category page loads with paginated product grid (24 products) |
| 3 | Scrolls, taps product card "Blue Cotton Kurti" | SCR-001 | Interested | SSR PDP renders with images from CloudFront |
| 4 | Reviews images, price ₹999, description | SCR-001 | Evaluating | Image carousel responds to swipe; In Stock badge visible |
| 5 | Taps "Add to Cart" | SCR-001 | Committed | Cart badge increments; success toast: "Added to cart" |
| 6 | Navigates to cart | SCR-002 | Focused | Cart shows product, qty, subtotal |
| 7 | Confirms saved address is correct | SCR-002 Checkout | Relieved | Pre-filled address with edit option |
| 8 | Taps "Pay" → Razorpay modal | SCR-002 | Slightly anxious | Razorpay Checkout.js modal opens |
| 9 | Completes UPI payment | External (Razorpay) | Anxious → Relieved | Payment confirmed by webhook |
| 10 | Sees order confirmation | SCR-002 Success | Satisfied | Order ID, expected delivery, confirmation email sent |

### Error Path: Item Sells Out Between Add-to-Cart and Checkout

| Step | Trigger | System Response | Recovery |
|------|---------|----------------|---------|
| 6 | Stock hits 0 between cart and checkout | "This item is now out of stock. Removed from cart." | Cart updates; "Continue Shopping" CTA shown |

### Edge Case: Seller's Own Product

| Trigger | Response |
|---------|---------|
| Rahul (also a seller) views his own product's PDP | "Add to Cart" and "Buy Now" disabled; alert: "You cannot purchase your own product" (BR-012) |

---

## Journey 2: Anjali Purchases via WhatsApp Link (Guest Checkout)

**Entry Point:** Seller shares `shopnest.in/products/{slug}` on WhatsApp  
**Exit Point:** Order placed; tracking URL in hand  
**Screens touched:** SCR-001 (PDP) → SCR-002 (Guest Checkout → Success + tracking token)

### Happy Path

| Step | Action | Screen | Emotional State | System Response |
|------|--------|--------|----------------|----------------|
| 1 | Taps WhatsApp link | Loading → SCR-001 | Curious, no prior ShopNest context | SSR renders product immediately; no login required |
| 2 | Reviews product — no login wall | SCR-001 | Comfortable | All info visible to guest |
| 3 | Taps "Buy Now" | SCR-001 | Decided | Redirected to guest checkout form |
| 4 | Fills guest form: name, email, phone, address | SCR-002 Guest | Engaged | Inline blur validation; phone auto-formats +91 |
| 5 | Submits form; reviews order summary | SCR-002 | Checking | Shows product, price, delivery estimate |
| 6 | Taps "Continue to Payment" → Razorpay | SCR-002 | Slightly anxious | Razorpay modal opens |
| 7 | Pays via UPI | External | Anxious → Relieved | Payment captured |
| 8 | Sees success page with tracking URL | SCR-002 Success | Satisfied | Tracking URL displayed + copy button + emailed to her |

### Error Path: Invalid Phone at Form Blur

| Step | Trigger | Response |
|------|---------|---------|
| 4 | Enters 9-digit phone | Inline error on blur: "Enter a valid 10-digit Indian mobile number" |
| — | Corrects to 10 digits | Error clears; field turns green |

### Edge Case: Link to Inactive/Pending Product

| Trigger | Response |
|---------|---------|
| Product is Pending Review, Rejected, or Seller Suspended | HTTP 404: "This product is no longer available" — no status reason leaked |

---

## Journey 3: Anjali Tracks and Cancels an Order (Guest)

**Entry Point:** Order confirmation email → tracking URL  
**Exit Point:** Order cancelled; refund initiated  
**Screens touched:** SCR-008 (Guest Order Tracking)

### Happy Path — Tracking

| Step | Action | Screen | Emotional State | System Response |
|------|--------|--------|----------------|----------------|
| 1 | Opens tracking URL from email | SCR-008 | Curious, slightly anxious | HMAC-verified URL; no login required |
| 2 | Sees order status, product list, seller | SCR-008 | Informed | Current status clearly shown (Payment Confirmed → Processing → Shipped → Delivered) |
| 3 | After seller marks Shipped: sees AWB + courier | SCR-008 | Relieved | AWB number + courier name visible |

### Happy Path — Cancellation (Pre-confirmation)

| Step | Action | Screen | System Response |
|------|--------|--------|----------------|
| 1 | Opens tracking URL | SCR-008 | — | Status: "Payment Confirmed" — seller has not confirmed yet |
| 2 | Taps "Cancel Order" | SCR-008 | Uncertain | Confirmation modal: "Are you sure? You'll receive a full refund." |
| 3 | Confirms cancellation | SCR-008 | Committed | POST /api/v1/orders/{id}/cancel → Razorpay refund initiated |
| 4 | Status updates to "Cancelled" | SCR-008 | Resolved | "Refund will appear in 3–7 business days" message shown |

### Error Path: Cancellation After Seller Confirms

| Trigger | Response |
|---------|---------|
| Order status = "Processing" (seller confirmed) | "Cancel Order" button replaced with message: "Cancellation not available — seller is processing your order" (BR-002) |

---

## Journey 4: Rahul Browses Search Results

**Entry Point:** Search bar on any buyer page  
**Exit Point:** Product Detail Page  
**Screens touched:** SCR-006 (Search Results) → SCR-001 (PDP)

### Happy Path

| Step | Action | Screen | Emotional State | System Response |
|------|--------|--------|----------------|----------------|
| 1 | Types "cotton kurti" in search bar, submits | SCR-006 | Goal-directed | Search results load (PostgreSQL FTS, P95 < 500ms) |
| 2 | Scans results grid: images, names, prices | SCR-006 | Evaluating | Ranked by relevance; 24 per page |
| 3 | Taps product card | SCR-001 | Interested | PDP loads |

### Error Path: No Results

| Trigger | Response |
|---------|---------|
| Search returns 0 results | "No products found for 'xyz'" + suggested categories + "Clear search" CTA |

---

## Journey 5: Rahul Views His Order History

**Entry Point:** "My Orders" from buyer account menu  
**Exit Point:** Order tracking page or product re-purchase  
**Screens touched:** SCR-009 (Buyer Order History) → SCR-008 / SCR-001

### Happy Path

| Step | Action | Screen | Emotional State | System Response |
|------|--------|--------|----------------|----------------|
| 1 | Taps profile → "My Orders" | SCR-009 | Task-focused | Order list loads (most recent first) |
| 2 | Sees all 5 orders with status badges | SCR-009 | Informed | Order ID, date, items, total, status |
| 3 | Taps an order for tracking detail | SCR-008 | — | Order detail / tracking page |

---

## Journey 6: Priya Registers and Sets Up Her Seller Account

**Entry Point:** Marketing landing page → Register as Seller  
**Exit Point:** Seller dashboard, ready to list products  
**Screens touched:** SCR-010 (Seller Registration) → SCR-011 (Store Setup Wizard) → SCR-012 (Subscription Signup) → SCR-013 (Dashboard)

### Happy Path

| Step | Action | Screen | Emotional State | System Response |
|------|--------|--------|----------------|----------------|
| 1 | Opens /register/seller | SCR-010 | Motivated, hopeful | Registration form: email, password, name, business name, phone, city |
| 2 | Submits registration | SCR-010 | Expectant | HTTP 201: account created, status "Pending Review" |
| 3 | Begins store setup wizard (Step 1: Store Info) | SCR-011 | Focused | Store name, logo upload, banner upload |
| 4 | Uploads logo (< 2MB), banner (< 5MB) | SCR-011 | Uploading | S3 upload; progress indicator; validation on file size/type |
| 5 | Selects 2 store categories | SCR-011 Step 2 | Quick | Multi-select from platform categories |
| 6 | Proceeds to subscription payment | SCR-012 | Committing | Shows plan: ₹1,999/month; features listed |
| 7 | Pays via Razorpay Subscriptions | SCR-012 | Slightly anxious → Relieved | Razorpay modal; subscription created; status → Active |
| 8 | Lands on seller dashboard | SCR-013 | Excited, ready to start | Empty dashboard with "Add Your First Product" CTA |
| 9 | Admin approves her seller application | (async) | Relieved (via email) | Email: "Your seller account is approved" |

### Error Path: Logo Too Large

| Trigger | Response |
|---------|---------|
| Logo file > 2MB | "Logo must be under 2MB. Please compress your image and try again." |

### Error Path: Subscription Payment Fails

| Trigger | Response |
|---------|---------|
| Razorpay returns payment failure | "Payment failed. No amount charged. Please try again." — Retry CTA shown |

---

## Journey 7: Priya Manages Products (Full Lifecycle)

**Entry Point:** Seller dashboard → Products  
**Exit Point:** Products active on marketplace, orders being received  
**Screens touched:** SCR-003 (Product Management) — existing + tested

### Happy Path (abbreviated — see Journey 3 in testing run for full detail)

| Step | Action | Result |
|------|--------|--------|
| 1 | Adds new product | Status → "Pending Review" |
| 2 | Admin approves | Email received; product → Active; visible on marketplace |
| 3 | Edits price/stock | Immediate; no re-review |
| 4 | Edits name/description | Triggers re-review; product temporarily hidden |

---

## Journey 8: Priya Manages Incoming Orders

**Entry Point:** Seller dashboard → Orders tab  
**Exit Point:** Orders fulfilled; awaiting weekly payout  
**Screens touched:** SCR-014 (Seller Order Management)

### Happy Path

| Step | Action | Screen | Emotional State | System Response |
|------|--------|--------|----------------|----------------|
| 1 | Opens Orders tab | SCR-014 | Task-oriented | Order list: newest first; each shows buyer (first name + last initial), items, total, status |
| 2 | Sees new order "Payment Confirmed" | SCR-014 | Excited | New order notification (email already sent) |
| 3 | Reviews order details | SCR-014 | Focused | Product, qty, buyer name, delivery address |
| 4 | Clicks "Confirm Order" | SCR-014 | Committed | Status → "Processing"; buyer can no longer cancel |
| 5 | Packs item, enters courier + AWB | SCR-014 | Working | Text fields for courier name + AWB |
| 6 | Clicks "Mark as Shipped" | SCR-014 | Satisfied | Status → "Shipped"; buyer receives shipping email with AWB |
| 7 | (Optional) Marks as Delivered | SCR-014 | Done | Status → "Delivered" |

### Error Path: Missing AWB

| Trigger | Response |
|---------|---------|
| Submits "Mark as Shipped" without AWB | "AWB number and courier name are required" — inline error |

---

## Journey 9: Admin Reviews Seller Application

**Entry Point:** Admin login → Seller Approval Queue  
**Exit Point:** Seller approved or rejected  
**Screens touched:** SCR-018 (Admin Login) → SCR-019 (Admin Seller Approval Queue)

### Happy Path — Approval

| Step | Action | Screen | System Response |
|------|--------|--------|----------------|
| 1 | Admin logs in | SCR-018 | Admin dashboard loads |
| 2 | Opens Seller Queue | SCR-019 | Pending applications listed newest first |
| 3 | Clicks application row to expand | SCR-019 | Seller details: name, business, email, city, registration date |
| 4 | Clicks "Approve" | SCR-019 | Seller status → Active; seller receives approval email; row moves to Approved tab |

### Happy Path — Rejection

| Step | Action | Screen | System Response |
|------|--------|--------|----------------|
| 3 | Reviews application; finds issues | SCR-019 | — |
| 4 | Clicks "Reject", enters reason | SCR-019 | Modal with text area |
| 5 | Confirms rejection | SCR-019 | Seller status → Rejected; email with reason sent |

---

## Journey 10: Admin Reviews Product Listing

**Entry Point:** Admin dashboard → Product Approval Queue  
**Exit Point:** Product approved or rejected  
**Screens touched:** SCR-020 (Admin Product Approval Queue)

### Happy Path — Approval

| Step | Action | Screen | System Response |
|------|--------|--------|----------------|
| 1 | Opens Product Queue | SCR-020 | Pending products listed |
| 2 | Clicks product to expand | SCR-020 | Product images, name, description, price, seller info shown |
| 3 | Approves | SCR-020 | Product → Active; visible on marketplace within 30s; seller notified by email |

### Happy Path — Rejection

| Step | Action | System Response |
|------|--------|----------------|
| 2 | Reviews product; finds issues | — |
| 3 | Enters rejection reason, confirms | Product → Rejected; reason stored; seller email + dashboard inline display |

---

## Moments of Friction Analysis

| Journey | Highest Friction Point | Designed Resolution |
|---------|----------------------|---------------------|
| Journey 1 (Rahul purchase) | Checkout address entry if new user | Saved address pre-fill; add new address inline |
| Journey 2 (Anjali guest) | Phone number format uncertainty | +91 prefix visible; inputmode="numeric"; error on blur |
| Journey 3 (Anjali tracking) | Finding tracking URL (in email, not saved anywhere) | Copy-to-clipboard on success page; prominent email reminder |
| Journey 6 (Priya registration) | Logo/banner upload on mobile | File size + type shown before upload; progress bar per file; image preview |
| Journey 8 (Priya orders) | Knowing when to confirm vs. when order auto-confirms | Explicit "Confirm Order" CTA with tooltip: "Buyer can cancel until you confirm" |
| Journey 9 (Admin seller review) | Not seeing seller contact info for questions | Email shown in expanded row; contact admin action available |
| Journey 10 (Admin product review) | Judging product quality from small thumbnail | Images expandable to full size in modal on click |
