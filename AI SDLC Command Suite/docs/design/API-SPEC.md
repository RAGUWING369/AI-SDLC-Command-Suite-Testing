# API Specification — ShopNest
**Phase:** 04 — Architecture & Design
**Generated:** 2026-05-14
**Status:** Draft — Awaiting Human Gate Approval
**Style:** RESTful, OpenAPI 3.0 compatible
**Base URL:** `https://api.shopnest.in/api/v1`
**Auth:** RS256-signed JWT Bearer token (except public endpoints)
**Format:** `Content-Type: application/json` — snake_case fields throughout

---

## Global Conventions

### Authentication
All authenticated endpoints require:
```
Authorization: Bearer <access_token>
```
Public endpoints (marketplace browse, order tracking, webhooks) do not require auth.

### Standard Error Response
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human-readable description",
    "details": { "field_name": ["error message"] }
  }
}
```

### Standard HTTP Status Codes
| Code | Meaning |
|------|---------|
| 200 | OK — successful read or update |
| 201 | Created — resource created |
| 202 | Accepted — async task queued (analytics, webhook) |
| 204 | No Content — successful delete |
| 400 | Bad Request — validation error |
| 401 | Unauthorized — missing or expired JWT |
| 403 | Forbidden — valid JWT but wrong role or subscription status |
| 404 | Not Found |
| 409 | Conflict — duplicate resource (email, razorpay_order_id) |
| 422 | Unprocessable Entity — business rule violation |
| 429 | Too Many Requests — rate limit exceeded |
| 500 | Internal Server Error |

### Versioning Policy
- Current version: `v1` (prefix: `/api/v1/`)
- Deprecation notice minimum: 6 months before removal
- Deprecated endpoints return `Sunset` and `Deprecation` response headers

### Pagination
List endpoints use cursor-based pagination:
```json
{
  "results": [...],
  "next": "https://api.shopnest.in/api/v1/resource?cursor=xxx",
  "previous": null,
  "count": 47
}
```
Default page size: 20. Maximum: 100. Query param: `?limit=20&cursor=xxx`

---

## Rate Limiting

| Endpoint Group | Limit | Window | Response on Exceeded |
|---------------|-------|--------|---------------------|
| POST /auth/login | 10 | per minute per IP | 429 + Retry-After header |
| POST /sellers/register | 5 | per minute per IP | 429 |
| POST /buyers/register | 10 | per minute per IP | 429 |
| POST /analytics/track | 100 | per session per minute | 429 (silent — analytics never block UI) |
| All other endpoints | 300 | per minute per authenticated user | 429 |

---

## 1. Authentication

### POST /api/v1/sellers/register
**Auth:** None | **Role:** Public

**Request:**
```json
{
  "email": "priya@mynaturals.in",
  "password": "Secure1234",
  "full_name": "Priya Sharma",
  "business_name": "My Naturals",
  "store_name": "My Naturals Store",
  "phone": "9876543210",
  "city": "Mumbai",
  "gstin": "27AABCU9603R1ZX",
  "shipping_pincode": "400001"
}
```
**Validation:** email unique (409 if duplicate), email blocked (409 if registration_blocked=true), password ≥8 chars + 1 digit, phone 10 digits, pincode 6 digits, gstin format if provided.

**Response 201:**
```json
{
  "id": "uuid",
  "email": "priya@mynaturals.in",
  "status": "PENDING_REVIEW",
  "message": "Registration successful. Your account is pending admin review."
}
```
**Errors:** 409 (duplicate email or blocked email), 400 (validation)

---

### POST /api/v1/buyers/register
**Auth:** None | **Role:** Public

**Request:**
```json
{
  "email": "rahul@gmail.com",
  "password": "Secure5678",
  "full_name": "Rahul Mehta",
  "phone": "9123456789"
}
```
**Response 201:**
```json
{
  "id": "uuid",
  "email": "rahul@gmail.com",
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "Bearer",
  "access_expires_in": 86400,
  "refresh_expires_in": 2592000
}
```
**Errors:** 409 (duplicate email), 400 (validation)

---

### POST /api/v1/auth/login
**Auth:** None | **Role:** Public

**Request:**
```json
{
  "email": "priya@mynaturals.in",
  "password": "Secure1234"
}
```
**Response 200:**
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "Bearer",
  "access_expires_in": 86400,
  "refresh_expires_in": 2592000,
  "user": {
    "id": "uuid",
    "email": "priya@mynaturals.in",
    "role": "SELLER",
    "seller_status": "ACTIVE"
  }
}
```
**Errors:** 401 (invalid credentials), 403 (account inactive or suspended)

---

### POST /api/v1/auth/refresh
**Auth:** None (refresh token in body) | **Role:** Public

**Request:**
```json
{ "refresh_token": "eyJ..." }
```
**Response 200:**
```json
{
  "access_token": "eyJ...",
  "access_expires_in": 86400
}
```
**Errors:** 401 (invalid or expired refresh token)

---

### POST /api/v1/auth/logout
**Auth:** Bearer token | **Role:** Any authenticated

**Request:**
```json
{ "refresh_token": "eyJ..." }
```
**Response 204** — refresh token invalidated (added to denylist in Redis)

---

## 2. Seller Profile & Store

### GET /api/v1/seller/me
**Auth:** Seller JWT | **Role:** Seller (any status)

**Response 200:**
```json
{
  "id": "uuid",
  "full_name": "Priya Sharma",
  "business_name": "My Naturals",
  "phone": "9876543210",
  "city": "Mumbai",
  "gstin": "27AABCU9603R1ZX",
  "shipping_pincode": "400001",
  "status": "ACTIVE",
  "store": {
    "id": "uuid",
    "name": "My Naturals Store",
    "slug": "my-naturals-store",
    "logo_cdn_url": "https://cdn.shopnest.in/...",
    "setup_complete": true
  },
  "subscription": {
    "status": "ACTIVE",
    "current_end": "2026-06-05T00:00:00Z"
  }
}
```

---

### GET /api/v1/seller/store
**Auth:** Seller JWT | **Role:** Seller (Active + Active subscription)

**Response 200:**
```json
{
  "id": "uuid",
  "name": "My Naturals Store",
  "slug": "my-naturals-store",
  "logo_cdn_url": "https://cdn.shopnest.in/...",
  "banner_cdn_url": "https://cdn.shopnest.in/...",
  "categories": [
    { "id": "uuid", "name": "Skincare", "slug": "skincare" }
  ],
  "setup_complete": true
}
```

---

### PATCH /api/v1/seller/store
**Auth:** Seller JWT | **Role:** Seller (Active + Active subscription)

**Request:** `multipart/form-data`
```
name: "My Naturals Store Updated"
logo: <file: PNG/JPG, max 2MB>
banner: <file: PNG/JPG, max 5MB>
category_ids: ["uuid1", "uuid2"]
shipping_pincode: "400002"
```
**Response 200:** Updated store object (same schema as GET)
**Errors:** 400 (validation), 422 (MIME type invalid, file too large)

---

### PATCH /api/v1/seller/payout-details
**Auth:** Seller JWT | **Role:** Seller (any subscription status per FR-SELLER-007)

**Request:**
```json
{
  "account_holder_name": "Priya Sharma",
  "account_number": "12345678901234",
  "ifsc_code": "HDFC0001234"
}
```
**Response 200:**
```json
{
  "id": "uuid",
  "account_holder_name": "Priya Sharma",
  "ifsc_code": "HDFC0001234",
  "account_number_masked": "**********1234",
  "updated_at": "2026-05-14T09:00:00Z"
}
```
> **Note:** Account number is never returned in full. Masked form only.

---

### GET /api/v1/seller/subscription
**Auth:** Seller JWT | **Role:** Seller

**Response 200:**
```json
{
  "id": "uuid",
  "status": "ACTIVE",
  "amount_paise": 199900,
  "current_start": "2026-05-05T00:00:00Z",
  "current_end": "2026-06-05T00:00:00Z",
  "paid_count": 1,
  "razorpay_subscription_id": "sub_XXXX",
  "grace_period_ends_at": null
}
```

---

### GET /api/v1/seller/invoices
**Auth:** Seller JWT | **Role:** Seller

**Response 200:**
```json
{
  "results": [
    {
      "id": "uuid",
      "invoice_number": "INV-2026-000001",
      "amount_paise": 199900,
      "gst_amount_paise": 35982,
      "issued_at": "2026-05-05T12:00:00Z",
      "download_url": "https://api.shopnest.in/api/v1/seller/invoices/uuid/download"
    }
  ],
  "count": 1,
  "next": null
}
```

---

## 3. Products — Seller

### POST /api/v1/seller/products
**Auth:** Seller JWT | **Role:** Seller (Active + Active subscription)

**Request:** `multipart/form-data`
```
name: "Aloe Vera Face Wash" (max 200 chars)
description: "Natural..." (max 2000 chars)
price_paise: 29900
stock_quantity: 50
category_id: "uuid"
images[0]: <file: JPEG/PNG, max 5MB>
images[1]: <file: JPEG/PNG, max 5MB>
```
**Response 201:**
```json
{
  "id": "uuid",
  "name": "Aloe Vera Face Wash",
  "status": "PENDING_REVIEW",
  "price_paise": 29900,
  "stock_quantity": 50,
  "images": [
    { "id": "uuid", "cdn_url": "https://cdn.shopnest.in/...", "display_order": 1 }
  ],
  "created_at": "2026-05-14T10:00:00Z"
}
```
**Errors:** 400 (validation), 403 (seller not active or no subscription), 422 (MIME invalid, too many images)

---

### GET /api/v1/seller/products
**Auth:** Seller JWT | **Role:** Seller (Active + Active subscription)

**Query params:** `?status=ACTIVE&category_id=uuid&page=1&limit=20`

**Response 200:** Paginated list of seller's products with same fields as POST response.

---

### GET /api/v1/seller/products/{product_id}
**Auth:** Seller JWT | **Role:** Seller (owns the product)

**Response 200:** Full product object including all images and rejection_reason if REJECTED.

---

### PATCH /api/v1/seller/products/{product_id}
**Auth:** Seller JWT | **Role:** Seller (owns the product)

**Request:** `multipart/form-data` (any subset of fields)
```
name: "Updated Name"       ← triggers re-review (BR-017)
description: "Updated..."  ← triggers re-review (BR-017)
price_paise: 27900         ← does NOT trigger re-review
stock_quantity: 45         ← does NOT trigger re-review
images[0]: <file>          ← triggers re-review (BR-017)
```
**Response 200:** Updated product. Status may change to PENDING_REVIEW if name/description/images changed.

---

### DELETE /api/v1/seller/products/{product_id}
**Auth:** Seller JWT | **Role:** Seller (owns the product)

**Response 204** — Product status set to DELISTED (soft delete).
**Errors:** 422 if product has active orders (BR-009)

---

## 4. Orders — Seller

### GET /api/v1/seller/orders
**Auth:** Seller JWT | **Role:** Seller (Active)

**Query params:** `?fulfillment_status=PENDING&limit=20&cursor=xxx`

**Response 200:**
```json
{
  "results": [
    {
      "id": "uuid",
      "order_id": "uuid",
      "fulfillment_status": "PENDING",
      "subtotal_paise": 59800,
      "item_count": 2,
      "buyer_city": "Mumbai",
      "created_at": "2026-05-14T08:30:00Z"
    }
  ],
  "count": 3,
  "next": null
}
```

---

### GET /api/v1/seller/orders/{sub_order_id}
**Auth:** Seller JWT | **Role:** Seller (owns the sub-order)

**Response 200:**
```json
{
  "id": "uuid",
  "order_id": "uuid",
  "fulfillment_status": "PENDING",
  "subtotal_paise": 59800,
  "delivery_address": {
    "name": "Rahul Mehta",
    "phone": "9123456789",
    "line1": "123 MG Road",
    "city": "Mumbai",
    "state": "Maharashtra",
    "pincode": "400001"
  },
  "line_items": [
    {
      "id": "uuid",
      "product_name_snapshot": "Aloe Vera Face Wash",
      "unit_price_paise": 29900,
      "quantity": 2,
      "total_paise": 59800
    }
  ],
  "created_at": "2026-05-14T08:30:00Z"
}
```

---

### POST /api/v1/seller/orders/{sub_order_id}/confirm
**Auth:** Seller JWT | **Role:** Seller (owns the sub-order)

**Response 200:**
```json
{
  "id": "uuid",
  "fulfillment_status": "PROCESSING",
  "confirmed_at": "2026-05-14T09:15:00Z"
}
```
**Errors:** 422 if sub-order is not in PENDING status

---

### POST /api/v1/seller/orders/{sub_order_id}/ship
**Auth:** Seller JWT | **Role:** Seller (owns sub-order)

**Request:**
```json
{
  "awb_number": "BD123456789IN",
  "shipping_carrier": "BlueDart"
}
```
**Response 200:**
```json
{
  "id": "uuid",
  "fulfillment_status": "SHIPPED",
  "awb_number": "BD123456789IN",
  "shipping_carrier": "BlueDart",
  "shipped_at": "2026-05-15T14:00:00Z"
}
```
**Side effect:** Triggers async SES email to buyer with AWB number.
**Errors:** 422 if not in PROCESSING status

---

### POST /api/v1/seller/orders/{sub_order_id}/deliver
**Auth:** Seller JWT | **Role:** Seller (owns sub-order)

**Response 200:**
```json
{
  "id": "uuid",
  "fulfillment_status": "DELIVERED",
  "delivered_at": "2026-05-17T16:00:00Z"
}
```
**Errors:** 422 if not in SHIPPED status

---

### GET /api/v1/seller/orders/export
**Auth:** Seller JWT | **Role:** Seller (Active)

**Query params:** `?start_date=2026-05-01&end_date=2026-05-31`

**Response 200:** `Content-Type: text/csv; charset=utf-8-sig`
CSV columns: `order_id, date, product_name, quantity, unit_price_inr, buyer_city, status`

---

## 5. Payouts — Seller

### GET /api/v1/seller/payouts
**Auth:** Seller JWT | **Role:** Seller

**Response 200:**
```json
{
  "results": [
    {
      "id": "uuid",
      "total_amount_paise": 342500,
      "status": "PROCESSED",
      "transfer_mode": "NEFT",
      "payout_date": "2026-05-12",
      "created_at": "2026-05-12T09:00:00Z"
    }
  ],
  "count": 3,
  "next": null
}
```

---

### GET /api/v1/seller/payouts/{payout_id}
**Auth:** Seller JWT | **Role:** Seller (owns the payout)

**Response 200:** Payout detail including all `SettlementLedgerEntry` records included in this payout.
```json
{
  "id": "uuid",
  "total_amount_paise": 342500,
  "status": "PROCESSED",
  "payout_date": "2026-05-12",
  "razorpay_payout_id": "pout_XXXX",
  "ledger_entries": [
    {
      "sub_order_id": "uuid",
      "gross_amount_paise": 59800,
      "gateway_fee_paise": 1314,
      "net_amount_paise": 58486
    }
  ]
}
```

---

## 6. Seller Analytics

### GET /api/v1/seller/analytics/summary
**Auth:** Seller JWT | **Role:** Seller (Active)

**Query params:** `?period=7d` or `?period=30d`

**Response 200:**
```json
{
  "period": "7d",
  "total_revenue_paise": 342500,
  "order_count": 12,
  "average_order_value_paise": 28542
}
```

---

### GET /api/v1/seller/analytics/top-products
**Auth:** Seller JWT | **Role:** Seller (Active)

**Query params:** `?period=30d&limit=5`

**Response 200:**
```json
{
  "results": [
    {
      "product_id": "uuid",
      "product_name": "Aloe Vera Face Wash",
      "order_count": 34,
      "total_revenue_paise": 1016600
    }
  ]
}
```

---

## 7. Marketplace — Public (Buyer-Facing)

### GET /api/v1/marketplace/products
**Auth:** None | **Role:** Public
**Business rule:** Returns only products where seller.status=ACTIVE AND subscription.status=ACTIVE AND product.status=ACTIVE (BR-001 Triple Gate).

**Query params:**
```
?q=aloe+vera            ← full-text search (PG FTS)
&category_id=uuid       ← filter by category
&min_price=10000        ← filter by min price (paise)
&max_price=100000       ← filter by max price (paise)
&sort=price_asc|price_desc|newest
&limit=20&cursor=xxx
```
**Response 200:**
```json
{
  "results": [
    {
      "id": "uuid",
      "name": "Aloe Vera Face Wash",
      "price_paise": 29900,
      "stock_quantity": 48,
      "primary_image_cdn_url": "https://cdn.shopnest.in/...",
      "store": {
        "id": "uuid",
        "name": "My Naturals Store",
        "slug": "my-naturals-store"
      },
      "category": { "id": "uuid", "name": "Skincare" }
    }
  ],
  "count": 127,
  "next": "...?cursor=abc"
}
```

---

### GET /api/v1/marketplace/products/{product_id}
**Auth:** None | **Role:** Public

**Response 200:**
```json
{
  "id": "uuid",
  "name": "Aloe Vera Face Wash",
  "description": "Natural aloe vera...",
  "price_paise": 29900,
  "stock_quantity": 48,
  "images": [
    { "cdn_url": "https://cdn.shopnest.in/...", "display_order": 1 },
    { "cdn_url": "https://cdn.shopnest.in/...", "display_order": 2 }
  ],
  "store": {
    "id": "uuid",
    "name": "My Naturals Store",
    "slug": "my-naturals-store",
    "logo_cdn_url": "https://cdn.shopnest.in/..."
  },
  "category": { "id": "uuid", "name": "Skincare", "slug": "skincare" }
}
```
**Errors:** 404 if product is not Active/visible

---

### GET /api/v1/marketplace/categories
**Auth:** None | **Role:** Public

**Response 200:**
```json
{
  "results": [
    { "id": "uuid", "name": "Skincare", "slug": "skincare", "product_count": 342 }
  ]
}
```

---

### GET /api/v1/marketplace/categories/{category_slug}/products
**Auth:** None | **Role:** Public

Identical response to `GET /marketplace/products` filtered by category. Supports same sort and price filter params. BR-001 Triple Gate enforced.

---

### GET /api/v1/marketplace/stores/{store_slug}
**Auth:** None | **Role:** Public

**Response 200:**
```json
{
  "id": "uuid",
  "name": "My Naturals Store",
  "slug": "my-naturals-store",
  "logo_cdn_url": "https://cdn.shopnest.in/...",
  "banner_cdn_url": "https://cdn.shopnest.in/...",
  "categories": [{ "id": "uuid", "name": "Skincare" }],
  "products": { "results": [...], "count": 47, "next": null }
}
```
**Errors:** 404 if store's seller is not Active or subscription is not Active

---

## 8. Cart

Cart is Redis-backed. Session UUID is set by the backend as an HTTP-only cookie on first request. Cart merges on buyer login (guest cart → registered buyer cart).

### GET /api/v1/cart
**Auth:** Optional Bearer token | **Role:** Guest or Buyer
**Session:** Identified by `X-Session-ID` header or `session_id` cookie

**Response 200:**
```json
{
  "session_id": "uuid",
  "items": [
    {
      "product_id": "uuid",
      "product_name": "Aloe Vera Face Wash",
      "price_paise": 29900,
      "quantity": 2,
      "total_paise": 59800,
      "primary_image_cdn_url": "https://cdn.shopnest.in/...",
      "stock_available": 48,
      "seller_id": "uuid"
    }
  ],
  "item_count": 2,
  "subtotal_paise": 59800
}
```

---

### POST /api/v1/cart/items
**Auth:** Optional | **Role:** Guest or Buyer

**Request:**
```json
{
  "product_id": "uuid",
  "quantity": 2
}
```
**Response 201:** Updated cart (same schema as GET)
**Errors:** 422 if quantity > stock_available, 422 if product not Active, 422 if buyer is the seller of this product (BR-012)

---

### PATCH /api/v1/cart/items/{product_id}
**Auth:** Optional | **Role:** Guest or Buyer

**Request:** `{ "quantity": 3 }` (set to 0 to remove)

**Response 200:** Updated cart

---

### DELETE /api/v1/cart/items/{product_id}
**Auth:** Optional | **Role:** Guest or Buyer

**Response 200:** Updated cart with item removed

---

### DELETE /api/v1/cart
**Auth:** Optional | **Role:** Guest or Buyer

**Response 204** — cart cleared

---

## 9. Checkout & Payments

### POST /api/v1/checkout/initiate
**Auth:** Optional (guest or buyer) | **Role:** Guest or Buyer

Creates a Razorpay order and validates the cart.

**Request:**
```json
{
  "session_id": "uuid",
  "delivery_address": {
    "name": "Rahul Mehta",
    "phone": "9123456789",
    "line1": "123 MG Road",
    "line2": "",
    "city": "Mumbai",
    "state": "Maharashtra",
    "pincode": "400001"
  },
  "guest_email": "rahul.guest@gmail.com"
}
```
> `guest_email` required only if unauthenticated (guest buyer).

**Response 201:**
```json
{
  "order_id": "uuid",
  "razorpay_order_id": "order_XXXX",
  "razorpay_key_id": "rzp_live_XXXX",
  "amount_paise": 59800,
  "currency": "INR",
  "prefill": {
    "name": "Rahul Mehta",
    "email": "rahul.guest@gmail.com",
    "contact": "9123456789"
  }
}
```
**Side effects:** Creates `Order` record (status=PENDING); validates stock; creates Razorpay order.
**Errors:** 422 (stock changed since cart load, out-of-stock), 422 (self-purchase: seller buying own product BR-012)

---

### POST /api/v1/checkout/verify
**Auth:** Optional | **Role:** Guest or Buyer

Called by frontend after Razorpay widget closes successfully (before webhook arrives).

**Request:**
```json
{
  "razorpay_order_id": "order_XXXX",
  "razorpay_payment_id": "pay_XXXX",
  "razorpay_signature": "hex_signature"
}
```
**Response 200:**
```json
{
  "order_id": "uuid",
  "tracking_token": "hmac_token",
  "status": "CAPTURED",
  "message": "Payment confirmed. Order placed successfully."
}
```
> **Note:** Frontend verification is a UX fast-path. The authoritative payment confirmation is the `payment.captured` webhook (handled separately). Stock decrement happens via webhook, not this endpoint.

---

### POST /api/v1/checkout/razorpay-webhook
**Auth:** HMAC-SHA256 `X-Razorpay-Signature` header validated by WebhookValidatorMiddleware
**Role:** Razorpay server (IP allowlist + signature)

**Handles events:**
- `payment.captured` → decrement stock (atomic), update Order status=CAPTURED, create SubOrder records, create SettlementLedgerEntry records, enqueue order confirmation emails (Celery), enqueue audit log entry
- `payment.failed` → update Order status=FAILED, release held stock

**Request body:** Razorpay standard webhook payload
**Response 200:** `{ "status": "ok" }` — must respond 200 within 5s (processing is async via Celery)
**Errors:** 400 (invalid signature — logged to AuditLog)

---

### POST /api/v1/subscriptions/razorpay-webhook
**Auth:** HMAC-SHA256 `X-Razorpay-Signature` header
**Role:** Razorpay server

**Handles events:**
- `subscription.charged` → update Subscription status=ACTIVE, increment paid_count, clear grace_period_ends_at, generate GST invoice (Celery), send invoice email
- `subscription.charge.failed` → set grace_period_ends_at = now + 7 days, send failure notification email, enqueue suspension check job

**Response 200:** `{ "status": "ok" }`

---

## 10. Orders — Buyer

### GET /api/v1/orders/{order_id}
**Auth:** Optional (guest uses tracking_token param) | **Role:** Buyer or Guest (with token)

**Query param:** `?tracking_token=hmac_token` (guest only; not required for authenticated buyer)

**Response 200:**
```json
{
  "id": "uuid",
  "payment_status": "CAPTURED",
  "total_amount_paise": 59800,
  "delivery_address": { ... },
  "sub_orders": [
    {
      "id": "uuid",
      "seller_name": "My Naturals Store",
      "fulfillment_status": "PROCESSING",
      "awb_number": null,
      "line_items": [...]
    }
  ],
  "created_at": "2026-05-14T09:00:00Z"
}
```
**Errors:** 403 (buyer JWT but different buyer), 404

---

### GET /api/v1/orders
**Auth:** Buyer JWT | **Role:** Registered Buyer only

**Query params:** `?limit=20&cursor=xxx`

**Response 200:** Paginated list of buyer's orders (same structure as GET /orders/{id} but summary fields only).

---

### POST /api/v1/orders/{order_id}/cancel
**Auth:** Optional (guest provides tracking_token in body) | **Role:** Buyer or Guest

**Request (guest):**
```json
{ "tracking_token": "hmac_token" }
```
**Business rule:** Only cancellable if ALL sub-orders are in PENDING fulfillment status (BR-002 — buyer cannot cancel after seller confirms).

**Response 200:**
```json
{
  "order_id": "uuid",
  "status": "REFUNDED",
  "message": "Order cancelled. Refund initiated to your original payment method."
}
```
**Side effects:** Calls Razorpay Refunds API, restores stock quantities (atomic), updates SettlementLedgerEntry to WITHHELD, sends cancellation email.
**Errors:** 422 if any sub-order is in PROCESSING/SHIPPED/DELIVERED status

---

### GET /api/v1/orders/track/{tracking_token}
**Auth:** None | **Role:** Public (token-authenticated)

Stateless guest order tracking. Validates HMAC-SHA256 token.

**Response 200:** Same as GET /orders/{order_id} response.
**Errors:** 404 (invalid or expired token)

---

## 11. Admin — Seller Management

All admin endpoints require `Platform Admin` role JWT.

### GET /api/v1/admin/sellers
**Auth:** Admin JWT | **Role:** Admin

**Query params:** `?status=PENDING_REVIEW&limit=20&cursor=xxx`

**Response 200:** Paginated list of sellers with status, registration date, store name, subscription status.

---

### GET /api/v1/admin/sellers/{seller_id}
**Auth:** Admin JWT | **Role:** Admin

**Response 200:** Full seller detail including store, subscription status, product count.

---

### POST /api/v1/admin/sellers/{seller_id}/approve
**Auth:** Admin JWT | **Role:** Admin

**Response 200:**
```json
{
  "seller_id": "uuid",
  "status": "ACTIVE",
  "approved_at": "2026-05-14T10:00:00Z"
}
```
**Side effects:** Sends approval email to seller via Celery.

---

### POST /api/v1/admin/sellers/{seller_id}/reject
**Auth:** Admin JWT | **Role:** Admin

**Request:**
```json
{ "rejection_reason": "Incomplete business documentation provided." }
```
**Response 200:**
```json
{
  "seller_id": "uuid",
  "status": "REJECTED",
  "registration_blocked": true
}
```
**Side effects:** Sets `registration_blocked=true` on User; sends rejection email with reason via Celery.

---

### POST /api/v1/admin/sellers/{seller_id}/suspend
**Auth:** Admin JWT | **Role:** Admin

**Request:**
```json
{ "reason": "Policy violation: counterfeit products listed." }
```
**Response 200:** `{ "seller_id": "uuid", "status": "SUSPENDED" }`
**Side effects:** Delists all seller's Active products; withholds pending payouts; logs AuditLog entry.

---

### POST /api/v1/admin/sellers/{seller_id}/reinstate
**Auth:** Admin JWT | **Role:** Admin

**Response 200:** `{ "seller_id": "uuid", "status": "ACTIVE" }`
**Side effects:** Restores seller's products to Active if subscription is Active; processes withheld payouts in next weekly batch.

---

## 12. Admin — Product Management

### GET /api/v1/admin/products
**Auth:** Admin JWT | **Role:** Admin

**Query params:** `?status=PENDING_REVIEW&seller_id=uuid&limit=20&cursor=xxx`

**Response 200:** Paginated list of products with name, seller store name, status, submission date, thumbnail.

---

### GET /api/v1/admin/products/{product_id}
**Auth:** Admin JWT | **Role:** Admin

**Response 200:** Full product detail including all images, description, seller info.

---

### POST /api/v1/admin/products/{product_id}/approve
**Auth:** Admin JWT | **Role:** Admin

**Response 200:**
```json
{
  "product_id": "uuid",
  "status": "ACTIVE",
  "approved_at": "2026-05-14T11:00:00Z"
}
```
**Side effects:** Product appears in marketplace (BR-001 triple gate checks pass); sends approval email to seller.

---

### POST /api/v1/admin/products/{product_id}/reject
**Auth:** Admin JWT | **Role:** Admin

**Request:**
```json
{ "rejection_reason": "Product images are low quality. Please upload clear photos." }
```
**Response 200:** `{ "product_id": "uuid", "status": "REJECTED" }`
**Side effects:** Sends rejection email with reason to seller.

---

### POST /api/v1/admin/products/{product_id}/delist
**Auth:** Admin JWT | **Role:** Admin

**Request:**
```json
{ "reason": "Reported as prohibited item." }
```
**Response 200:** `{ "product_id": "uuid", "status": "DELISTED" }`
**Side effects:** Product immediately disappears from marketplace; logs AuditLog entry.

---

## 13. Admin — Dashboard

### GET /api/v1/admin/dashboard
**Auth:** Admin JWT | **Role:** Admin

**Response 200:**
```json
{
  "active_sellers": 87,
  "pending_seller_approvals": 4,
  "pending_product_approvals": 12,
  "total_products_active": 1243,
  "orders_last_30d": 342,
  "gmv_last_30d_paise": 8934200,
  "mrr_paise": 173913,
  "payment_success_rate_7d": 0.982
}
```

---

## 14. Analytics

### POST /api/v1/analytics/track
**Auth:** Optional (anonymous events accepted without auth) | **Role:** Any

Fire-and-forget event tracking. Always returns 202 regardless of whether the event was written successfully (analytics must never block the UI).

**Request:**
```json
{
  "event_name": "checkout.payment_completed",
  "session_id": "uuid",
  "entity_type": "Order",
  "entity_id": "uuid",
  "properties": {
    "order_value_inr_paise": 59800,
    "payment_method": "upi",
    "buyer_type": "guest"
  },
  "platform": "web_buyer",
  "device_type": "mobile"
}
```
**Response 202:** `{ "status": "queued" }`
**Rate limit:** 100 events/session/minute

---

## 15. Health Check

### GET /api/v1/health/
**Auth:** None | **Role:** Public (used by ALB health check)

**Response 200:**
```json
{
  "status": "ok",
  "db": "ok",
  "cache": "ok",
  "version": "1.0.0"
}
```
**Response 503:** If DB or Redis is unreachable

---

## Endpoint Summary Table

| # | Method | Path | Auth | Role |
|---|--------|------|------|------|
| 1 | POST | /auth/sellers/register | None | Public |
| 2 | POST | /auth/buyers/register | None | Public |
| 3 | POST | /auth/login | None | Public |
| 4 | POST | /auth/refresh | None | Public |
| 5 | POST | /auth/logout | Bearer | Any |
| 6 | GET | /seller/me | Bearer | Seller |
| 7 | GET | /seller/store | Bearer | Seller |
| 8 | PATCH | /seller/store | Bearer | Seller (Active) |
| 9 | PATCH | /seller/payout-details | Bearer | Seller |
| 10 | GET | /seller/subscription | Bearer | Seller |
| 11 | GET | /seller/invoices | Bearer | Seller |
| 12 | POST | /seller/products | Bearer | Seller (Active) |
| 13 | GET | /seller/products | Bearer | Seller (Active) |
| 14 | GET | /seller/products/{id} | Bearer | Seller (owns) |
| 15 | PATCH | /seller/products/{id} | Bearer | Seller (owns) |
| 16 | DELETE | /seller/products/{id} | Bearer | Seller (owns) |
| 17 | GET | /seller/orders | Bearer | Seller (Active) |
| 18 | GET | /seller/orders/{id} | Bearer | Seller (owns) |
| 19 | POST | /seller/orders/{id}/confirm | Bearer | Seller (owns) |
| 20 | POST | /seller/orders/{id}/ship | Bearer | Seller (owns) |
| 21 | POST | /seller/orders/{id}/deliver | Bearer | Seller (owns) |
| 22 | GET | /seller/orders/export | Bearer | Seller (Active) |
| 23 | GET | /seller/payouts | Bearer | Seller |
| 24 | GET | /seller/payouts/{id} | Bearer | Seller (owns) |
| 25 | GET | /seller/analytics/summary | Bearer | Seller (Active) |
| 26 | GET | /seller/analytics/top-products | Bearer | Seller (Active) |
| 27 | GET | /marketplace/products | None | Public |
| 28 | GET | /marketplace/products/{id} | None | Public |
| 29 | GET | /marketplace/categories | None | Public |
| 30 | GET | /marketplace/categories/{slug}/products | None | Public |
| 31 | GET | /marketplace/stores/{slug} | None | Public |
| 32 | GET | /cart | Optional | Guest/Buyer |
| 33 | POST | /cart/items | Optional | Guest/Buyer |
| 34 | PATCH | /cart/items/{product_id} | Optional | Guest/Buyer |
| 35 | DELETE | /cart/items/{product_id} | Optional | Guest/Buyer |
| 36 | DELETE | /cart | Optional | Guest/Buyer |
| 37 | POST | /checkout/initiate | Optional | Guest/Buyer |
| 38 | POST | /checkout/verify | Optional | Guest/Buyer |
| 39 | POST | /checkout/razorpay-webhook | HMAC | Razorpay |
| 40 | POST | /subscriptions/razorpay-webhook | HMAC | Razorpay |
| 41 | GET | /orders/{order_id} | Optional | Buyer/Guest |
| 42 | GET | /orders | Bearer | Buyer |
| 43 | POST | /orders/{order_id}/cancel | Optional | Buyer/Guest |
| 44 | GET | /orders/track/{token} | None | Public |
| 45 | GET | /admin/sellers | Bearer | Admin |
| 46 | GET | /admin/sellers/{id} | Bearer | Admin |
| 47 | POST | /admin/sellers/{id}/approve | Bearer | Admin |
| 48 | POST | /admin/sellers/{id}/reject | Bearer | Admin |
| 49 | POST | /admin/sellers/{id}/suspend | Bearer | Admin |
| 50 | POST | /admin/sellers/{id}/reinstate | Bearer | Admin |
| 51 | GET | /admin/products | Bearer | Admin |
| 52 | GET | /admin/products/{id} | Bearer | Admin |
| 53 | POST | /admin/products/{id}/approve | Bearer | Admin |
| 54 | POST | /admin/products/{id}/reject | Bearer | Admin |
| 55 | POST | /admin/products/{id}/delist | Bearer | Admin |
| 56 | GET | /admin/dashboard | Bearer | Admin |
| 57 | POST | /analytics/track | Optional | Any |
| 58 | GET | /health/ | None | Public |

**Total: 58 endpoints** across 11 resource groups. All endpoints are documented; all are traceable to user stories in USER-STORIES.md.
