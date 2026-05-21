# PRD Analytics Plan — ShopNest
**Phase:** 03 — Product Requirements Document
**Generated:** 2026-05-05
**Status:** Draft — Awaiting Human Gate Approval
**Source Authority:** PRD.md §8; REQUIREMENTS.md §FR-ANALYTICS; SUCCESS-METRICS.md

---

> **Purpose:** This document specifies the full analytics instrumentation plan for ShopNest MVP. It defines every event to track, the properties to capture with each event, the funnels to monitor, the dashboards to build, the privacy compliance mapping, and the implementation responsibilities per layer (frontend vs. backend).
>
> **Audience:** Phase 4 (Architecture), Phase 7 (Implementation), Phase 9 (Testing — analytics validation), Phase 13 (Monitoring — dashboard setup).
>
> **Privacy baseline:** ShopNest is subject to India's Digital Personal Data Protection Act (DPDPA) 2023. No Personally Identifiable Information (PII) must appear in any analytics event property. All event data is internal-only (ShopNest's analytics database); no third-party behavioral analytics SDK (e.g., Mixpanel, Segment, Amplitude) is instrumented at MVP — all events are captured to a ShopNest-owned `analytics_events` PostgreSQL table and surfaced via an internal admin dashboard.

---

## 1. Analytics Architecture

### 1.1 Data Flow

```
Browser / Mobile (Next.js)              Django Backend
       │                                      │
       │  POST /api/v1/analytics/track        │
       │─────────────────────────────────────►│
       │                                      │
       │                               analytics_events
       │                               PostgreSQL table
       │                                      │
                                       Admin Dashboard
                                       (Phase 13 setup)
```

### 1.2 Analytics Event Table Schema

```sql
CREATE TABLE analytics_events (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_name    VARCHAR(100) NOT NULL,           -- Namespaced event name
    user_id       UUID REFERENCES users(id),       -- NULL for anonymous
    session_id    UUID NOT NULL,                   -- Assigned at session start
    entity_type   VARCHAR(50),                     -- e.g. 'product', 'order', 'seller'
    entity_id     UUID,                            -- FK to relevant entity (if applicable)
    properties    JSONB NOT NULL DEFAULT '{}',     -- Event-specific properties (no PII)
    platform      VARCHAR(20) NOT NULL,            -- 'web_buyer', 'web_seller', 'web_admin'
    device_type   VARCHAR(20),                     -- 'mobile', 'tablet', 'desktop'
    created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes for dashboard query performance
CREATE INDEX idx_analytics_events_name ON analytics_events (event_name);
CREATE INDEX idx_analytics_events_created_at ON analytics_events (created_at DESC);
CREATE INDEX idx_analytics_events_session ON analytics_events (session_id);
CREATE INDEX idx_analytics_events_entity ON analytics_events (entity_type, entity_id);
```

### 1.3 API Endpoint

```
POST /api/v1/analytics/track
Content-Type: application/json
Authorization: Bearer <jwt>  (optional — anonymous events accepted without auth)

{
  "event_name": "<namespaced_event_name>",
  "session_id": "<uuid>",
  "entity_type": "<type>",
  "entity_id": "<uuid>",
  "properties": { ... },
  "platform": "web_buyer",
  "device_type": "mobile"
}

Response: 202 Accepted (fire-and-forget; analytics must never block the critical path)
```

**Implementation notes:**
- Analytics endpoint must respond within 50ms (write to queue, not directly to DB)
- Use Django Channels or Celery task for async DB write
- Failed analytics writes must NOT propagate errors to the frontend
- Rate-limited to 100 events/session/minute to prevent abuse

---

## 2. Event Taxonomy

### Naming Convention

All events use dot-separated namespace format: `[area].[action]`

| Area Prefix | Description |
|-------------|-------------|
| `session` | Session lifecycle |
| `buyer` | Buyer account actions |
| `seller` | Seller account and dashboard actions |
| `product` | Product browsing events |
| `search` | Search interactions |
| `cart` | Cart management events |
| `checkout` | Checkout funnel events |
| `order` | Order lifecycle events |
| `subscription` | Seller subscription events |
| `payout` | Seller payout events |
| `admin` | Platform admin actions |
| `notification` | Email notification delivery events |

---

### 2.1 Session Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `session.started` | New session initialized (first page load or API call) | Backend (middleware) | P1 |
| `session.ended` | Session expired or user explicitly logs out | Backend (middleware) | P2 |

**`session.started` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `session_id` | UUID | `"a1b2c3..."` | No |
| `referrer_source` | String | `"organic"`, `"direct"`, `"referral"` | No |
| `user_type` | String | `"guest"`, `"buyer"`, `"seller"`, `"admin"` | No |
| `device_type` | String | `"mobile"`, `"tablet"`, `"desktop"` | No |
| `is_returning` | Boolean | `true` | No |

---

### 2.2 Buyer Account Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `buyer.registered` | Buyer completes registration (email verified) | Backend | P2 |
| `buyer.logged_in` | Buyer authentication success | Backend | P2 |
| `buyer.login_failed` | Authentication failure | Backend | P2 |
| `buyer.password_reset_requested` | Password reset email requested | Backend | P3 |

**`buyer.registered` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `registration_method` | String | `"email"` | No |
| `has_prior_guest_cart` | Boolean | `true` | No |

---

### 2.3 Seller Account & Dashboard Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `seller.registration_started` | Seller begins registration form | Frontend | P1 |
| `seller.registration_submitted` | Seller submits complete registration form | Backend | P1 |
| `seller.registration_approved` | Admin approves seller account | Backend | P1 |
| `seller.registration_rejected` | Admin rejects seller account | Backend | P1 |
| `seller.store_setup_completed` | Seller completes store profile (name, logo, bank details) | Backend | P1 |
| `seller.dashboard_viewed` | Seller views dashboard home | Frontend | P2 |
| `seller.orders_page_viewed` | Seller views orders list | Frontend | P2 |
| `seller.bank_details_updated` | Seller updates bank account details | Backend | P2 |
| `seller.suspended` | Seller account suspended by admin | Backend | P1 |
| `seller.reinstated` | Seller account reinstated by admin | Backend | P1 |

**`seller.registration_submitted` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `has_gstin` | Boolean | `false` | No |
| `business_type` | String | `"sole_proprietor"`, `"partnership"`, `"pvt_ltd"` | No |
| `time_to_submit_seconds` | Integer | `420` | No |

**`seller.registration_approved` / `seller.registration_rejected` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `days_since_submission` | Float | `1.5` | No |
| `rejection_reason_category` | String | `"incomplete_docs"` (rejected only) | No |

---

### 2.4 Product Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `product.listed` | Seller submits new product listing | Backend | P1 |
| `product.approved` | Admin approves product listing | Backend | P1 |
| `product.rejected` | Admin rejects product listing | Backend | P1 |
| `product.edited` | Seller edits product (price, stock, or re-review fields) | Backend | P2 |
| `product.delisted` | Seller or admin delists product | Backend | P2 |
| `product.viewed` | Buyer views a product detail page | Frontend | P1 |
| `product.image_viewed` | Buyer clicks/swipes to view a product image | Frontend | P3 |
| `product.low_stock_alert_triggered` | Stock drops to ≤5 units | Backend | P2 |

**`product.listed` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `product_id` | UUID | `"d4e5f6..."` | No |
| `category_id` | UUID | `"a1b2c3..."` | No |
| `price_inr` | Integer (paise) | `29900` | No |
| `stock_quantity` | Integer | `50` | No |
| `image_count` | Integer | `3` | No |
| `has_description` | Boolean | `true` | No |

**`product.viewed` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `product_id` | UUID | `"d4e5f6..."` | No |
| `seller_id` | UUID | `"s1a2b3..."` | No |
| `category_id` | UUID | `"a1b2c3..."` | No |
| `price_inr` | Integer (paise) | `29900` | No |
| `source` | String | `"search"`, `"category"`, `"homepage_featured"` | No |
| `position_in_list` | Integer | `3` | No |
| `stock_available` | Boolean | `true` | No |

**`product.approved` / `product.rejected` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `product_id` | UUID | `"d4e5f6..."` | No |
| `seller_id` | UUID | `"s1a2b3..."` | No |
| `hours_in_review` | Float | `18.5` | No |
| `rejection_reason_category` | String | `"prohibited_item"` (rejected only) | No |

---

### 2.5 Search Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `search.query_submitted` | Buyer submits a search query | Frontend + Backend | P1 |
| `search.result_clicked` | Buyer clicks a product in search results | Frontend | P1 |
| `search.no_results` | Search returns zero results | Backend | P1 |
| `search.filter_applied` | Buyer applies category or price filter to search | Frontend | P2 |

**`search.query_submitted` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `query_hash` | String (SHA-256 of query, not raw query) | `"3a4b5c..."` | No |
| `query_length_chars` | Integer | `12` | No |
| `results_count` | Integer | `47` | No |
| `response_time_ms` | Integer | `210` | No |
| `filters_active` | Boolean | `false` | No |

> **Privacy note:** Raw search query text is NOT stored. The SHA-256 hash enables duplicate detection and frequency ranking without capturing potentially PII-containing queries (e.g., a buyer who searches for their own name).

**`search.result_clicked` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `product_id` | UUID | `"d4e5f6..."` | No |
| `position_in_results` | Integer | `2` | No |
| `query_hash` | String | `"3a4b5c..."` | No |
| `results_count` | Integer | `47` | No |

---

### 2.6 Cart Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `cart.item_added` | Buyer adds a product to cart | Frontend + Backend | P1 |
| `cart.item_removed` | Buyer removes a product from cart | Frontend | P2 |
| `cart.quantity_changed` | Buyer updates product quantity in cart | Frontend | P2 |
| `cart.viewed` | Buyer opens the cart page / drawer | Frontend | P2 |
| `cart.abandoned` | Cart has items but session expires (TTL 24h) | Backend (Celery) | P1 |
| `cart.merged` | Guest cart merged with registered buyer cart at login | Backend | P2 |

**`cart.item_added` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `product_id` | UUID | `"d4e5f6..."` | No |
| `seller_id` | UUID | `"s1a2b3..."` | No |
| `category_id` | UUID | `"a1b2c3..."` | No |
| `price_inr` | Integer (paise) | `29900` | No |
| `quantity` | Integer | `2` | No |
| `cart_size_before` | Integer | `1` | No |
| `source` | String | `"product_detail"`, `"search"`, `"category"` | No |
| `buyer_type` | String | `"guest"`, `"registered"` | No |

**`cart.abandoned` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `cart_item_count` | Integer | `3` | No |
| `cart_value_inr` | Integer (paise) | `89700` | No |
| `sellers_count` | Integer | `2` | No |
| `time_in_cart_hours` | Float | `14.5` | No |
| `buyer_type` | String | `"guest"`, `"registered"` | No |

---

### 2.7 Checkout Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `checkout.started` | Buyer clicks "Proceed to Checkout" | Frontend | P1 |
| `checkout.address_submitted` | Buyer submits delivery address | Frontend | P1 |
| `checkout.payment_initiated` | Razorpay checkout widget opened | Frontend | P1 |
| `checkout.payment_completed` | `payment.captured` webhook received | Backend | P1 |
| `checkout.payment_failed` | `payment.failed` webhook received | Backend | P1 |
| `checkout.order_created` | Order record created in DB post-payment | Backend | P1 |
| `checkout.self_purchase_blocked` | Buyer attempted to buy own product (BR-012) | Backend | P2 |

**`checkout.started` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `cart_item_count` | Integer | `3` | No |
| `cart_value_inr` | Integer (paise) | `89700` | No |
| `sellers_count` | Integer | `2` | No |
| `buyer_type` | String | `"guest"`, `"registered"` | No |

**`checkout.payment_completed` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `order_id` | UUID | `"o1a2b3..."` | No |
| `order_value_inr` | Integer (paise) | `89700` | No |
| `payment_method` | String | `"upi"`, `"card"`, `"netbanking"`, `"emi"`, `"wallet"` | No |
| `sellers_count` | Integer | `2` | No |
| `item_count` | Integer | `3` | No |
| `buyer_type` | String | `"guest"`, `"registered"` | No |
| `time_from_checkout_start_seconds` | Integer | `87` | No |

**`checkout.payment_failed` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `razorpay_error_code` | String | `"BAD_REQUEST_ERROR"` | No |
| `razorpay_error_reason` | String | `"payment_failed"` | No |
| `payment_method` | String | `"upi"` | No |
| `cart_value_inr` | Integer (paise) | `89700` | No |
| `buyer_type` | String | `"guest"`, `"registered"` | No |

---

### 2.8 Order Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `order.confirmed_by_seller` | Seller confirms a sub-order | Backend | P1 |
| `order.shipped` | Seller marks sub-order as shipped (AWB entered) | Backend | P1 |
| `order.delivered` | Seller marks sub-order as delivered | Backend | P2 |
| `order.cancelled_by_buyer` | Buyer cancels order (before seller confirmation) | Backend | P1 |
| `order.tracking_viewed` | Buyer views order tracking page | Frontend | P2 |
| `order.confirmation_email_sent` | SES dispatches order confirmation email | Backend | P1 |
| `order.shipping_email_sent` | SES dispatches shipping notification email | Backend | P2 |

**`order.confirmed_by_seller` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `sub_order_id` | UUID | `"so1a2b3..."` | No |
| `seller_id` | UUID | `"s1a2b3..."` | No |
| `hours_since_payment` | Float | `3.5` | No |
| `item_count` | Integer | `2` | No |
| `sub_order_value_inr` | Integer (paise) | `59800` | No |

**`order.cancelled_by_buyer` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `order_id` | UUID | `"o1a2b3..."` | No |
| `order_value_inr` | Integer (paise) | `89700` | No |
| `minutes_since_payment` | Float | `18.0` | No |
| `buyer_type` | String | `"guest"`, `"registered"` | No |

---

### 2.9 Subscription Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `subscription.plan_viewed` | Seller views subscription pricing page | Frontend | P1 |
| `subscription.signup_initiated` | Seller clicks "Subscribe Now" | Frontend | P1 |
| `subscription.activated` | `subscription.charged` webhook received (first charge) | Backend | P1 |
| `subscription.renewed` | `subscription.charged` webhook received (renewal) | Backend | P1 |
| `subscription.payment_failed` | `subscription.charge.failed` webhook received | Backend | P1 |
| `subscription.grace_period_entered` | 7-day grace period begins post-failure | Backend | P1 |
| `subscription.account_suspended` | Account suspended post-grace-period | Backend | P1 |
| `subscription.reactivated` | Seller successfully recovers from suspension | Backend | P1 |
| `subscription.cancelled` | Seller cancels subscription | Backend | P2 |

**`subscription.activated` / `subscription.renewed` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `seller_id` | UUID | `"s1a2b3..."` | No |
| `plan_amount_inr` | Integer (paise) | `199900` | No |
| `billing_cycle` | String | `"monthly"` | No |
| `razorpay_subscription_id` | String | `"sub_xxxx"` | No |
| `is_first_charge` | Boolean | `true` | No |

**`subscription.payment_failed` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `seller_id` | UUID | `"s1a2b3..."` | No |
| `failure_count` | Integer | `1` | No |
| `razorpay_error_code` | String | `"PAYMENT_FAILED"` | No |
| `days_until_suspension` | Integer | `7` | No |

---

### 2.10 Payout Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `payout.batch_run_started` | Weekly Celery job begins | Backend | P1 |
| `payout.batch_run_completed` | Weekly Celery job completes | Backend | P1 |
| `payout.seller_payout_initiated` | Individual seller payout transfer started | Backend | P1 |
| `payout.seller_payout_completed` | Razorpay Payouts API confirms success | Backend | P1 |
| `payout.seller_payout_failed` | Razorpay Payouts API returns error | Backend | P1 |
| `payout.seller_payout_skipped` | Seller skipped (suspended or no eligible orders) | Backend | P2 |

**`payout.batch_run_completed` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `batch_date` | Date | `"2026-06-01"` | No |
| `sellers_paid` | Integer | `87` | No |
| `sellers_skipped` | Integer | `4` | No |
| `sellers_failed` | Integer | `1` | No |
| `total_disbursed_inr` | Integer (paise) | `2847600` | No |
| `duration_seconds` | Integer | `42` | No |

**`payout.seller_payout_failed` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `seller_id` | UUID | `"s1a2b3..."` | No |
| `amount_inr` | Integer (paise) | `34500` | No |
| `razorpay_error_code` | String | `"INVALID_IFSC"` | No |
| `will_retry` | Boolean | `false` | No |

---

### 2.11 Admin Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `admin.seller_approved` | Admin approves seller registration | Backend | P1 |
| `admin.seller_rejected` | Admin rejects seller registration | Backend | P1 |
| `admin.seller_suspended` | Admin manually suspends seller | Backend | P1 |
| `admin.seller_reinstated` | Admin reinstates suspended seller | Backend | P1 |
| `admin.product_approved` | Admin approves product listing | Backend | P1 |
| `admin.product_rejected` | Admin rejects product listing | Backend | P1 |
| `admin.product_delisted` | Admin delists approved product | Backend | P2 |
| `admin.platform_metrics_viewed` | Admin views platform analytics dashboard | Frontend | P3 |

---

### 2.12 Notification Events

| Event Name | Trigger | Implementation Layer | Priority |
|------------|---------|---------------------|---------|
| `notification.order_confirmation_sent` | SES order confirmation email dispatched to buyer | Backend | P1 |
| `notification.order_shipped_sent` | SES shipping notification email dispatched to buyer | Backend | P2 |
| `notification.new_order_alert_sent` | SES new order alert email dispatched to seller | Backend | P1 |
| `notification.payout_notification_sent` | SES payout notification email dispatched to seller | Backend | P2 |
| `notification.subscription_failed_sent` | SES subscription failure alert dispatched to seller | Backend | P1 |
| `notification.ses_bounce` | SES bounce webhook received | Backend | P2 |
| `notification.ses_complaint` | SES complaint webhook received | Backend | P2 |

**`notification.ses_bounce` / `notification.ses_complaint` properties:**

| Property | Type | Example | PII? |
|----------|------|---------|------|
| `email_type` | String | `"order_confirmation"` | No |
| `bounce_type` | String | `"Permanent"`, `"Transient"` (bounce only) | No |
| `recipient_type` | String | `"buyer"`, `"seller"` | No |

> **Privacy note:** The recipient email address is NOT stored in analytics events. Only `recipient_type` (buyer/seller) is captured.

---

## 3. Funnel Definitions

### Funnel 1: Guest Checkout Funnel
**Business goal:** Guest checkout completion >60%
**Measurement:** Percentage of sessions that reach `checkout.order_created` after `cart.item_added`

| Step | Event | Drop-off Point Label |
|------|-------|---------------------|
| 1 | `cart.item_added` | Entry (baseline) |
| 2 | `cart.viewed` | Abandoned before viewing cart |
| 3 | `checkout.started` | Abandoned after cart view |
| 4 | `checkout.address_submitted` | Abandoned at address form |
| 5 | `checkout.payment_initiated` | Abandoned before payment |
| 6 | `checkout.payment_completed` | Payment failed / abandoned |
| 7 | `checkout.order_created` | **Conversion** |

**Segmentation dimensions:** `buyer_type` (guest vs. registered), `device_type`, `payment_method`, `sellers_count` in cart

**Target metric:** Step 1 → Step 7 conversion: >60% for guest buyer sessions

---

### Funnel 2: Seller Onboarding Funnel
**Business goal:** Seller activation rate >70%; 100 paying sellers in 6 months
**Measurement:** Percentage of sellers who complete the full activation flow

| Step | Event | Drop-off Point Label |
|------|-------|---------------------|
| 1 | `seller.registration_started` | Entry (baseline) |
| 2 | `seller.registration_submitted` | Abandoned registration form |
| 3 | `seller.registration_approved` | Awaiting admin approval (or rejected) |
| 4 | `subscription.signup_initiated` | Approved but not subscribed |
| 5 | `subscription.activated` | Subscribed but store not set up |
| 6 | `seller.store_setup_completed` | **Fully Activated** |
| 7 | `product.listed` | Store set up but no products |

**Target metric:** Step 1 → Step 6: >70% activation rate
**Secondary metric:** Step 6 → Step 7: >80% (store setup completion target, NFR-USA-004)
**Admin review SLA metric:** Median hours between Step 2 and Step 3 (target: <24h)

---

### Funnel 3: Product Discovery-to-Purchase Funnel
**Business goal:** Buyer cart-to-order conversion >3%
**Measurement:** Percentage of product view sessions that result in an order

| Step | Event | Drop-off Point Label |
|------|-------|---------------------|
| 1 | `product.viewed` | Entry (baseline) |
| 2 | `cart.item_added` | Viewed but not added to cart |
| 3 | `checkout.started` | Added to cart but not checked out |
| 4 | `checkout.order_created` | **Conversion** |

**Segmentation dimensions:** `source` (search, category, homepage_featured), `device_type`, `category_id`, `price_inr` bucket

**Target metric:** Step 1 → Step 4: >3% conversion rate

---

## 4. Dashboard Specifications

> **Implementation:** Admin dashboards are built in Phase 13 (Monitoring). This section specifies what each dashboard must display to provide data against success metrics. All dashboards query the `analytics_events` PostgreSQL table.

### 4.1 Platform Overview Dashboard

**Audience:** Platform Admin
**Refresh cadence:** Daily (not real-time for MVP)

| Widget | Metric | Source Events | Time Range |
|--------|--------|--------------|-----------|
| Active Paying Sellers | COUNT DISTINCT seller_id WHERE subscription.activated in period | `subscription.activated`, `subscription.cancelled` | Current / last 30d / trend |
| MRR | Active sellers × ₹1,999 | Subscription status join | Current month |
| New Sellers This Month | COUNT `seller.registration_approved` | `seller.registration_approved` | Current month |
| Seller Activation Rate | Steps 1→6 conversion (Funnel 2) | Multiple | Last 30 days |
| Subscription Churn Rate | Cancelled / (Active at period start) | `subscription.cancelled`, subscription status | Last 30 days |
| Total GMV (Gross Merchandise Value) | SUM `order_value_inr` from `checkout.order_created` | `checkout.order_created` | Current month / trend |
| Orders Placed | COUNT `checkout.order_created` | `checkout.order_created` | Current month / trend |
| Payment Success Rate | `checkout.payment_completed` / (`checkout.payment_completed` + `checkout.payment_failed`) | Both events | Last 7 days |
| Pending Approval Queue | COUNT sellers in Pending Review | System table query | Real-time |
| Pending Products Queue | COUNT products in Pending Review | System table query | Real-time |

---

### 4.2 Buyer Funnel Dashboard

**Audience:** Platform Admin
**Refresh cadence:** Daily

| Widget | Metric | Source Events | Time Range |
|--------|--------|--------------|-----------|
| Guest Checkout Funnel | Steps 1–7 (Funnel 1) with conversion rates | See Funnel 1 | Last 7d / 30d |
| Product-to-Purchase Funnel | Steps 1–4 (Funnel 3) with conversion rates | See Funnel 3 | Last 7d / 30d |
| Cart Abandonment Rate | `cart.abandoned` / `cart.item_added` (unique sessions) | Both events | Last 7 days |
| Top Search Queries (by hash frequency) | Most frequent `query_hash` values | `search.query_submitted` | Last 7 days |
| Zero-Result Searches | COUNT `search.no_results` | `search.no_results` | Last 7 days |
| Payment Method Mix | Distribution of `payment_method` values | `checkout.payment_completed` | Last 30 days |
| Guest vs. Registered Buyer Orders | Ratio by `buyer_type` | `checkout.order_created` | Last 30 days |
| Device Mix | Distribution of `device_type` | `session.started` | Last 30 days |

---

### 4.3 Seller Operations Dashboard

**Audience:** Platform Admin
**Refresh cadence:** Daily

| Widget | Metric | Source Events | Time Range |
|--------|--------|--------------|-----------|
| Order Fulfillment Rate (48h) | % of `order.confirmed_by_seller` within 48h of payment | `checkout.payment_completed` vs `order.confirmed_by_seller` | Last 30 days |
| Orders in Pending State >48h | COUNT sub-orders without `order.confirmed_by_seller` >48h | System table query | Real-time |
| Top Sellers by GMV | SUM `sub_order_value_inr` by `seller_id` | `order.confirmed_by_seller` | Current month |
| Product Approval Turnaround | Median `hours_in_review` | `product.approved`, `product.rejected` | Last 30 days |
| Rejection Rate by Reason | Distribution of `rejection_reason_category` | `product.rejected`, `seller.registration_rejected` | Last 30 days |
| Weekly Payout Summary | Sellers paid, total disbursed, failures | `payout.batch_run_completed` | Last 4 weeks |
| Payout Failure Rate | `payout.seller_payout_failed` / `payout.seller_payout_initiated` | Both events | Last 4 weeks |
| Subscription Payment Failure Rate | `subscription.payment_failed` / `subscription.renewed` | Both events | Last 30 days |
| Sellers in Grace Period | COUNT sellers with `subscription.grace_period_entered` not followed by `subscription.reactivated` | Both events | Current |

---

### 4.4 Product Performance Dashboard

**Audience:** Platform Admin
**Refresh cadence:** Daily

| Widget | Metric | Source Events | Time Range |
|--------|--------|--------------|-----------|
| Most Viewed Products | COUNT `product.viewed` by `product_id` | `product.viewed` | Last 7 days |
| View-to-Cart Rate by Category | `cart.item_added` / `product.viewed` by `category_id` | Both events | Last 30 days |
| Low-Stock Alerts Triggered | COUNT `product.low_stock_alert_triggered` | `product.low_stock_alert_triggered` | Last 7 days |
| Products Awaiting Approval | COUNT products in Pending Review | System table query | Real-time |
| Products Listed This Month | COUNT `product.listed` | `product.listed` | Current month |
| Top Categories by Views | COUNT `product.viewed` by `category_id` | `product.viewed` | Last 30 days |

---

## 5. Privacy Compliance Mapping (DPDPA 2023)

| Requirement | ShopNest Analytics Approach | Compliance Status |
|-------------|---------------------------|-------------------|
| No collection of PII without consent | Analytics events capture zero PII fields (no name, email, phone, address, IP) | ✅ Compliant by design |
| Buyer right to erasure | `user_id` is set to NULL on account deletion; historical events become anonymous | ✅ Compliant |
| Data minimization | Events capture only business-necessary properties; no raw query text stored | ✅ Compliant |
| Purpose limitation | Analytics data used solely for internal platform improvement; not sold or shared | ✅ Compliant |
| Data localization | `analytics_events` table is in AWS RDS ap-south-1 (Mumbai) — India-resident data | ✅ Compliant |
| Retention limits | Analytics events older than 7 years purged per BR-011 (financial records); behavioral events older than 2 years purged as they are not financial records | ✅ Policy defined |
| Third-party analytics SDKs | None used in MVP — all analytics are first-party and internal | ✅ Compliant |
| Search query privacy | Raw query text never stored; SHA-256 hash stored for frequency analysis only | ✅ Compliant |
| Consent for analytics | Analytics are operational/platform-improvement in nature, not behavioral advertising; no separate consent required under DPDPA for internal operational analytics | ✅ Compliant (confirm with legal pre-launch) |

---

## 6. Implementation Responsibilities

| Area | Responsible Layer | Phase | Notes |
|------|-----------------|-------|-------|
| `analytics_events` table + indexes | Backend (Django migration) | Phase 7 | Write migration in `analytics` Django app |
| `POST /api/v1/analytics/track` endpoint | Backend (DRF view) | Phase 7 | Must respond 202; write async via Celery |
| Frontend event instrumentation (all P1 events) | Frontend (Next.js custom hook) | Phase 7 | Create `useAnalytics()` hook wrapping the API call |
| Frontend event instrumentation (P2/P3 events) | Frontend (Next.js) | Phase 7 | Lower priority; can be added iteratively |
| Backend event instrumentation | Backend (Django signals + service layer) | Phase 7 | Use Django signals where possible |
| Celery async write | Backend | Phase 7 | Add `track_event` Celery task |
| `cart.abandoned` detection | Backend (Celery beat job) | Phase 7 | Celery beat hourly scan of TTL-expiring carts |
| Dashboard queries | Backend (admin dashboard views) | Phase 13 | Phase 13 builds on analytics foundation |
| Analytics validation tests | Backend (pytest) | Phase 9 | Assert events written on key user flows |
| DPDPA erasure — null user_id | Backend (account deletion flow) | Phase 7 | Part of buyer account deletion FR |
| Analytics retention purge | Backend (Celery beat job) | Phase 7 | 2-year behavioral / 7-year financial cutoff |

---

## 7. Event Priority Summary

| Priority | Total Events | Description |
|----------|-------------|-------------|
| **P1 — Must Have for MVP** | 38 events | Business-critical; directly feed success metric measurement and funnel tracking |
| **P2 — Should Have for MVP** | 18 events | Important operational metrics; implement after P1 if time permits |
| **P3 — Could Have** | 4 events | Nice-to-have instrumentation; defer if sprint capacity is tight |
| **Total** | **60 events** | Across 12 event namespaces |

**P1 events must be instrumented and validated in Phase 9 testing before Phase 13 (Monitoring) dashboards are built.** P2 and P3 events are validated as part of integration testing but are not blocking for go-live.

---

## 8. Analytics Testing Requirements (Phase 9)

The following analytics tests must pass in Phase 9 before go-live:

| Test ID | Description | Expected Outcome |
|---------|-------------|-----------------|
| TC-ANLX-001 | Complete guest checkout — verify all P1 checkout funnel events fire in order | Events `checkout.started`, `checkout.address_submitted`, `checkout.payment_initiated`, `checkout.payment_completed`, `checkout.order_created` all present in `analytics_events` for the session |
| TC-ANLX-002 | Complete seller onboarding — verify all P1 seller funnel events fire | Events `seller.registration_submitted`, `seller.registration_approved`, `subscription.activated`, `seller.store_setup_completed` all present for the seller |
| TC-ANLX-003 | Product view → cart add → abandon — verify `cart.abandoned` event fires after TTL | `cart.abandoned` event present in `analytics_events` after Redis TTL simulation |
| TC-ANLX-004 | Payment failure — verify `checkout.payment_failed` fires and no order is created | `checkout.payment_failed` present; `checkout.order_created` absent for the session |
| TC-ANLX-005 | Subscription payment failure — verify grace period and suspension events fire in sequence | `subscription.payment_failed` → `subscription.grace_period_entered` → `subscription.account_suspended` |
| TC-ANLX-006 | Payout batch run — verify batch start/complete events and per-seller events | `payout.batch_run_started`, one `payout.seller_payout_initiated` per eligible seller, `payout.batch_run_completed` |
| TC-ANLX-007 | Search zero results — verify `search.no_results` fires for an unmatched query | `search.no_results` present; `query_hash` populated; raw query NOT stored |
| TC-ANLX-008 | Analytics endpoint performance — verify 202 response in <50ms under load | P95 response time <50ms at 100 concurrent event submissions |
| TC-ANLX-009 | PII audit — verify no PII present in any event properties in test run | Zero matches for email regex, phone regex, or name patterns in `properties` JSONB column |
| TC-ANLX-010 | Account deletion — verify `user_id` nulled in historic events | After buyer account deletion, all events for that `user_id` show `user_id = NULL` |

---

*This document is the authoritative analytics instrumentation specification for ShopNest MVP.*
*Phase 4 (Architecture) must include the `analytics_events` table and `POST /api/v1/analytics/track` endpoint in the system design.*
*Phase 7 (Implementation) must instrument all P1 events before sprint review.*
*Phase 9 (Testing) must execute all TC-ANLX tests before marking analytics instrumentation as complete.*
*Phase 13 (Monitoring) builds admin dashboards on the foundation defined in Section 4.*
