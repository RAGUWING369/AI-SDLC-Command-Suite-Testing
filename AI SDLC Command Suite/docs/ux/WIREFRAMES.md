# WIREFRAMES.md — ShopNest Wireframe Specifications

**Phase:** 05 — UX Design (Full Production Run)
**Date:** 2026-05-15
**Screens Covered:** SCR-001 through SCR-021 (21 screens)
**Wireframe Reference Frame:** 1440px desktop (HTML files `min-width: 1280px`); mobile layouts documented in state specs below
**Personas:** Rahul (Buyer) · Anjali (Guest Buyer) · Priya (Seller) · Admin

> **HTML wireframe files:** `docs/visuals/ux/SCR-NNN-*.html` — desktop 1440px, all states as vertical sections, inline `.ann` annotations. Open in any browser.

---

## 1. Complete Sitemap

```
shopnest.in/
│
├── / (Homepage)                             ← SCR-004: Marketplace Homepage [SSR]
│
├── /auth/                                   ← SCR-007: Buyer Login / Register [CSR]
│   ├── /auth/login
│   └── /auth/register
│
├── /categories/{slug}/                      ← SCR-005: Category Listing [SSR]
│
├── /search?q={query}                        ← SCR-006: Search Results [SSR]
│
├── /products/{slug}-{uuid}                  ← SCR-001: Product Detail Page [SSR]
│
├── /cart                                    ← SCR-002: Cart [CSR]
├── /checkout                                ← SCR-002 cont.: Checkout [CSR]
│   └── /checkout/success                   ← SCR-002 cont.: Order Success [CSR]
│
├── /orders/                                 ← SCR-009: Buyer Order History [CSR, auth-gated]
└── /orders/track/{token}                    ← SCR-008: Guest Order Tracking [CSR, HMAC-gated]
│
└── /seller/                                 ← Seller Portal (auth-gated: Seller role)
    ├── /seller/register                     ← SCR-010: Seller Registration [CSR]
    ├── /seller/setup                        ← SCR-011: Store Setup Wizard [CSR]
    ├── /seller/subscribe                    ← SCR-012: Seller Subscription Signup [CSR]
    ├── /seller/dashboard                    ← SCR-013: Seller Dashboard [CSR]
    ├── /seller/products                     ← SCR-003: Seller Product Management [CSR]
    │   ├── /seller/products/new
    │   └── /seller/products/{id}/edit
    ├── /seller/orders                       ← SCR-014: Seller Order Management [CSR]
    ├── /seller/payouts                      ← SCR-015: Seller Payouts [CSR]
    ├── /seller/subscription                 ← SCR-016: Seller Subscription Mgmt [CSR]
    └── /seller/settings                     ← SCR-017: Seller Store Settings [CSR]
│
└── /admin/                                  ← Admin Portal (auth-gated: Admin role)
    ├── /admin/login                         ← SCR-018: Admin Login [CSR]
    ├── /admin/dashboard                     ← SCR-021: Admin Dashboard [CSR]
    ├── /admin/sellers                       ← SCR-019: Admin Seller Queue [CSR]
    └── /admin/products                      ← SCR-020: Admin Product Queue [CSR]
```

---

## 2. Navigation Architecture

### 2a. Three Navigation Shells

ShopNest uses three distinct navigation shells — one per user type. Shells never mix.

| Shell | Persona | Desktop | Mobile |
|-------|---------|---------|--------|
| **Buyer Shell** | Rahul, Anjali | Sticky top bar (64px): Logo · Categories · Search · Cart (badge) · Sign In/Avatar | Sticky top bar (56px): Logo · Search · Cart · Avatar |
| **Seller Shell** | Priya | Left sidebar (240px fixed) + sticky header bar (56px) | Bottom tab bar (56px) + mobile header |
| **Admin Shell** | Admin | Left sidebar (240px fixed) + sticky header bar (56px) | Not required (Admin = desktop-only) |

**Rationale:** Buyers are in discovery/purchase mode (top nav familiar from Amazon/Flipkart). Sellers and admins are in operational/management mode (left sidebar reduces context-switching across workflow tasks). A unified nav would compromise both.

### 2b. Buyer Navigation — Detail

**Top bar items (left → right):**
```
[ShopNest Logo] [Categories ▼] [Search bar — full width] [🛒 Cart (n)] [Sign In / Avatar ▼]
```
- Categories dropdown: Women's, Men's, Electronics, Home & Kitchen, Sports, Books
- Search: full-text input; submits to `/search?q={query}` on Enter
- Cart badge: Zustand cart count; updates optimistically
- Sign In: shows avatar + name when authenticated; dropdown: My Orders, Profile, Sign Out

### 2c. Seller Sidebar — Detail

| Position | Item | Route | Icon |
|----------|------|-------|------|
| 1 | Dashboard | `/seller/dashboard` | 🏠 |
| 2 | Products | `/seller/products` | 📦 |
| 3 | Orders | `/seller/orders` | 📋 |
| 4 | Payouts | `/seller/payouts` | 💰 |
| divider | — | — | — |
| 5 | Settings | `/seller/settings` | ⚙️ |
| 6 | Subscription | `/seller/subscription` | 💳 |

Subscription item shows badge: `● Active` (green) or `⚠ Expired` (red).

### 2d. Admin Sidebar — Detail

| Position | Item | Route | Icon |
|----------|------|-------|------|
| 1 | Dashboard | `/admin/dashboard` | 🏠 |
| 2 | Seller Queue | `/admin/sellers` | 👥 |
| 3 | Product Queue | `/admin/products` | 📦 |

---

## 3. Screen Inventory

| ID | Screen | Route | Shell | Persona | User Stories | Priority | HTML File |
|----|--------|-------|-------|---------|-------------|----------|-----------|
| SCR-001 | Product Detail Page | `/products/{slug}-{uuid}` | Buyer | Rahul, Anjali | US-009, US-010, US-011, US-016 | P0 | SCR-001-product-detail.html |
| SCR-002 | Cart & Checkout | `/cart` → `/checkout` | Buyer | Rahul, Anjali | US-013, US-014, US-015, US-016, US-017 | P0 | SCR-002-cart-checkout.html |
| SCR-003 | Seller Product Management | `/seller/products` | Seller | Priya | US-004, US-005, US-006 | P0 | SCR-003-seller-products.html |
| SCR-004 | Marketplace Homepage | `/` | Buyer | Rahul, Anjali | US-011, US-012 | P0 | SCR-004-marketplace-homepage.html |
| SCR-005 | Category Listing | `/categories/{slug}` | Buyer | Rahul | US-012, US-013 | P0 | SCR-005-category-listing.html |
| SCR-006 | Search Results | `/search?q={query}` | Buyer | Rahul | US-014 | P0 | SCR-006-search-results.html |
| SCR-007 | Buyer Auth (Login/Register) | `/auth/login`, `/auth/register` | Buyer | Rahul | US-025, US-026 | P0 | SCR-007-buyer-auth.html |
| SCR-008 | Order Tracking | `/orders/track/{token}` | Buyer | Anjali, Rahul | US-019, US-020 | P0 | SCR-008-order-tracking.html |
| SCR-009 | Buyer Order History | `/orders` | Buyer | Rahul | US-027 | P1 | SCR-009-buyer-orders.html |
| SCR-010 | Seller Registration | `/seller/register` | Seller | Priya | US-001 | P0 | SCR-010-seller-registration.html |
| SCR-011 | Store Setup Wizard | `/seller/setup` | Seller | Priya | US-002 | P0 | SCR-011-store-setup-wizard.html |
| SCR-012 | Seller Subscription Signup | `/seller/subscribe` | Seller | Priya | US-003 | P0 | SCR-012-seller-subscription.html |
| SCR-013 | Seller Dashboard | `/seller/dashboard` | Seller | Priya | US-007, US-028 | P1 | SCR-013-seller-dashboard.html |
| SCR-014 | Seller Order Management | `/seller/orders` | Seller | Priya | US-007, US-008, US-009, US-034 | P0 | SCR-014-seller-orders.html |
| SCR-015 | Seller Payouts | `/seller/payouts` | Seller | Priya | US-032 | P1 | SCR-015-seller-payouts.html |
| SCR-016 | Seller Subscription Management | `/seller/subscription` | Seller | Priya | US-036 | P1 | SCR-016-seller-subscription-mgmt.html |
| SCR-017 | Seller Store Settings | `/seller/settings` | Seller | Priya | US-031 | P1 | SCR-017-seller-store-settings.html |
| SCR-018 | Admin Login | `/admin/login` | Admin | Admin | — | P0 | SCR-018-admin-login.html |
| SCR-019 | Admin Seller Approval Queue | `/admin/sellers` | Admin | Admin | US-021, US-023, US-029 | P0 | SCR-019-admin-seller-queue.html |
| SCR-020 | Admin Product Approval Queue | `/admin/products` | Admin | Admin | US-005, US-022, US-030 | P0 | SCR-020-admin-product-queue.html |
| SCR-021 | Admin Dashboard | `/admin/dashboard` | Admin | Admin | US-035 | P1 | SCR-021-admin-dashboard.html |

---

## 4. SCR-001: Product Detail Page (PDP)

### Overview
- **Route:** `/products/{slug}-{uuid}` — SSR (Next.js App Router)
- **Auth:** No — public guest-visible page
- **API:** `GET /api/v1/products/{id}` · `POST /api/v1/cart/items`
- **NFRs:** LCP < 2.5s (mobile 4G) · CloudFront image < 1s

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Default — In Stock, Authenticated | Buyer views active product |
| 2 | Loading Skeleton | SSR hydration in progress |
| 3 | Out of Stock | stock_quantity = 0 |
| 4 | Guest Visitor | No JWT present |
| 5 | Self-Purchase Block | Seller views own product (BR-012) |
| 6 | Product Not Found / Inactive | 404 from API |

#### State 1: Default (In Stock, Authenticated)
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ ShopNest Logo │ Categories ▼ │ [── Search ──────────────────] │ 🛒 2 │ Rahul ▼ │
├─────────────────────────────────────────────────────────────────────┤
│ Women's Clothing > Kurtis > Blue Cotton Kurti                        │  ← breadcrumb
├──────────────────────────────┬──────────────────────────────────────┤
│ [Image carousel — 560px wide]│  Blue Cotton Kurti                   │
│                               │  ₹999.00                            │
│  [●○○ indicators]             │  ✅ In Stock                         │
│  [Thumb] [Thumb] [Thumb]      │                                      │
│  (row of 4 thumbs below)      │  Quantity: [−] 1 [+]               │
│                               │                                      │
│                               │  [    Add to Cart    ] ← blue outline│
│                               │  [      Buy Now      ] ← orange fill │
│                               │                                      │
│                               │  ▾ Product Description               │
│                               │    Handloom cotton kurti...          │
│                               │                                      │
│                               │  🏪 Sold by: Priya's Boutique       │
│                               │     Bangalore · Active since 2025   │
│                               │                                      │
│                               │  🚚 Seller ships · No returns MVP   │
└──────────────────────────────┴──────────────────────────────────────┘

Mobile (375px):
┌─────────────────────────────────┐
│ ← Back   ShopNest         🛒 2  │
├─────────────────────────────────┤
│ [Image Carousel — full 375px]    │
│ ● ○ ○ (dots)                    │
├─────────────────────────────────┤
│ Women's Clothing > Kurtis        │
│ Blue Cotton Kurti                │  h1 20px bold
│ ₹999.00   ✅ In Stock            │
├─────────────────────────────────┤
│ Quantity: [−] 1 [+]             │
│ [    Add to Cart    ]           │
│ [      Buy Now      ]           │
├─────────────────────────────────┤
│ ▾ Product Description           │
│ 🏪 Priya's Boutique · Bangalore │
│ 🚚 Seller ships                 │
└─────────────────────────────────┘
```

**Annotations:**
1. Price = `price_paise / 100` formatted with ₹ prefix and 2dp
2. Stock badge: >10 → green "In Stock"; 1–10 → amber "Only {n} left!"; 0 → State 3
3. Cart badge count updates optimistically via Zustand on "Add to Cart"
4. "Buy Now" = add to cart + navigate to `/checkout` in one action
5. Quantity stepper max = `stock_quantity` from API response

#### State 2: Loading Skeleton
```
Same layout — all content replaced by shimmer blocks:
- Image area: 560×560px (desktop) or 375px full-width (mobile) shimmer rectangle
- Title: 2 shimmer lines (80% width, 40% width)
- Price: 1 shimmer line (30% width)
- Stock badge: 80px shimmer chip
- Stepper and buttons: shimmer rectangles matching button dimensions
```

#### State 3: Out of Stock
```
Same as State 1 except:
- Badge: ❌ Out of Stock (red)
- Quantity stepper: opacity-40, pointer-events-none
- "Add to Cart" button: disabled, gray-200 background
- "Buy Now" button: disabled, gray-200 background
```

#### State 4: Guest Visitor
```
Identical to State 1. No change — guest can add to cart and buy.
Guest checkout form shown at /checkout step (SCR-002 State 5).
No login wall on PDP.
```

#### State 5: Self-Purchase Block
```
Same as State 1 except:
- "Add to Cart" and "Buy Now" buttons: disabled (gray)
- Alert below buttons: ⚠️ "You cannot purchase your own product" (amber banner)
- Matches BR-012; checked via JWT seller_id vs product.seller_id
```

#### State 6: Product Not Found / Inactive
```
┌─────────────────────────────────┐
│ [Buyer top nav]                  │
├─────────────────────────────────┤
│         📦                       │
│  This product is no longer       │
│  available                       │  ← HTTP 404 (no status reason leaked)
│                                  │
│  [  Browse Products  ]           │  ← → homepage
└─────────────────────────────────┘
```
Triggered when: product.status ≠ ACTIVE, or seller.status ≠ ACTIVE, or subscription.status ≠ ACTIVE (Triple-Gate).

---

## 5. SCR-002: Cart & Checkout

### Overview
- **Route:** `/cart` (cart) → `/checkout` (checkout) → `/checkout/success` (success)
- **Auth:** No — supports guest checkout
- **API:** `GET /api/v1/cart/` · `PUT /api/v1/cart/items/{id}` · `DELETE /api/v1/cart/items/{id}` · `POST /api/v1/orders/`

### States (10 total — HTML wireframe is definitive spec)

| # | State | Key Feature |
|---|-------|-------------|
| 1 | Cart Default | 2-col layout; CartItemRow; QuantityStepper; OrderSummary |
| 2 | Loading Skeleton | DOM-mirror of State 1 |
| 3 | Empty Cart | EmptyCartPanel; CTA → homepage |
| 4 | API Error | ErrorPanel; role="alert"; Retry |
| 5 | Guest Checkout Form | GuestCheckoutForm; inline blur validation |
| 6 | Auth Checkout | AddressSelector; saved address radio group |
| 7 | Razorpay Modal | ShopNest dimmed; Razorpay dialog on top |
| 8 | Order Success (Auth) | OrderSuccessPanel; clearCart() |
| 9 | Guest Success | GuestTrackingToken; copy button; upsell |
| 10 | Payment Failed | PaymentFailurePanel; cart preserved; Retry |

**For full layout details see:** `docs/visuals/ux/SCR-002-cart-checkout.html`

#### Form Validation Spec (State 5 — Guest Checkout)

| Field | Rule | Error Message |
|-------|------|---------------|
| Full Name | Required, 2–100 chars | "Please enter your full name" |
| Email | RFC 5322 format | "Please enter a valid email address" |
| Phone | 10 digits, starts 6–9 | "Enter a valid 10-digit Indian mobile number" |
| Address Line 1 | Required, 5–200 chars | "Please enter your delivery address" |
| City | Required | "Please enter your city" |
| State | Required (select) | "Please select a state" |
| PIN Code | 6 digits | "Enter a valid 6-digit PIN code" |

Validation fires on `blur`. Re-validates on `input` after first failed blur.

---

## 6. SCR-003: Seller Product Management

### Overview
- **Route:** `/seller/products` — CSR, auth-gated (Seller role + Active subscription)
- **API:** `GET /api/v1/seller/products/` · `POST /api/v1/seller/products/` · `PATCH /api/v1/seller/products/{id}/`

### States (8 total — HTML wireframe is definitive spec)

| # | State | Key Feature |
|---|-------|-------------|
| 1 | Product List Default | Product table; all 4 status variants; sidebar nav |
| 2 | Loading Skeleton | DOM-mirror of State 1 |
| 3 | No Products | EmptyProductsPanel; admin approval notice |
| 4 | API Error | ErrorPanel inside main; sidebar functional |
| 5 | Subscription Gate | SubscriptionGatePanel; Triple-Gate; Renew CTA |
| 6 | Add Product Form | AdminReviewBanner; full form with ImageUploader |
| 7 | Form Errors | FormErrorSummary; linked error fields; disabled submit |
| 8 | Edit Product | ReReviewWarningBanner; per-field re-review indicators |

**For full layout details see:** `docs/visuals/ux/SCR-003-seller-products.html`

#### Form Validation Spec (States 6–8)

| Field | Rule | Re-review Required? | Error |
|-------|------|---------------------|-------|
| Name | Required, 3–200 chars | ✅ Yes | "Product name is required (3–200 characters)" |
| Description | Required, ≥100 chars | ✅ Yes | "Description must be at least 100 characters" |
| Price | Required, ≥ ₹1 (100 paise) | ❌ No | "Price must be at least ₹1.00" |
| Stock | Required, ≥0, integer | ❌ No | "Stock quantity must be 0 or more" |
| Category | Required, select | ✅ Yes (new) | "Please select a category" |
| Images | ≥1, JPEG/PNG/WebP, ≤5MB each | ✅ Yes (change) | "Please upload at least one product image" |

---

## 7. SCR-004: Marketplace Homepage

### Overview
- **Route:** `/` — SSR (SEO critical)
- **Auth:** No — public
- **API:** `GET /api/v1/products/?featured=true` · `GET /api/v1/categories/`
- **NFRs:** LCP < 2.5s; CloudFront CDN for images

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Default — Content Loaded | SSR renders with featured products |
| 2 | Loading Skeleton | SSR streaming / hydration |
| 3 | Empty Featured Products | No active products yet (bootstrap) |

#### State 1: Default Homepage
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Buyer Top Nav Bar — Logo | Categories | Search | Cart | Sign In]   │
├─────────────────────────────────────────────────────────────────────┤
│ [Hero Banner — 1440×400px, full bleed]                              │
│  "Shop What You Love" · "Discover products from verified sellers"   │
│  [  Browse All  ] ← orange CTA                                      │
├─────────────────────────────────────────────────────────────────────┤
│ Browse Categories                                                    │
│ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐            │
│ │Women │ │ Men  │ │Elect.│ │Home  │ │Sport │ │Books │            │
│ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘            │
├─────────────────────────────────────────────────────────────────────┤
│ Featured Products                                          [View All]│
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│ │[Product] │ │[Product] │ │[Product] │ │[Product] │              │
│ │Name      │ │Name      │ │Name      │ │Name      │              │
│ │₹999.00   │ │₹1,499.00 │ │₹599.00   │ │₹2,199.00 │              │
│ │★ Active  │ │★ Active  │ │★ Active  │ │★ Active  │              │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘              │
├─────────────────────────────────────────────────────────────────────┤
│ [Footer: About | Seller Sign Up | Contact | Terms | Privacy]        │
└─────────────────────────────────────────────────────────────────────┘

Mobile (375px):
┌─────────────────────────────────┐
│ [ShopNest Logo]    [🔍] [🛒 2]  │
├─────────────────────────────────┤
│ [Hero Banner — 375×200px]       │
│ "Shop What You Love"            │
│ [  Browse All  ]                │
├─────────────────────────────────┤
│ Browse Categories               │
│ [Women] [Men] [Electr.]  → scroll
├─────────────────────────────────┤
│ Featured Products               │
│ [Product card] [Product card]   │  2-col grid
│ [Product card] [Product card]   │
│ [  View All Products  ]         │
├─────────────────────────────────┤
│ [Footer links]                  │
└─────────────────────────────────┘
```

**Annotations:**
1. Hero banner image served from CloudFront; placeholder shown during load
2. Category icons: emoji at MVP; SVG icons post-MVP
3. Product cards show: image (CloudFront), name (1 line truncated), price (₹ format), seller name
4. "View All" → `/categories/all` or `/search` with no query
5. Featured products = up to 8 products selected by `featured=true` flag (admin-set)
6. Category grid: horizontal scroll on mobile; 6-column grid on desktop

#### State 2: Loading Skeleton
```
Hero: full-width shimmer rectangle 400px tall
Categories: 6 circular shimmer chips
Products: 4 rectangular product card skeletons (2-col mobile, 4-col desktop)
```

#### State 3: Empty Featured Products
```
Hero banner present; category grid present
Featured products section: "No featured products yet — check back soon!"
[Browse All Products] CTA visible
```

---

## 8. SCR-005: Category Listing

### Overview
- **Route:** `/categories/{slug}` — SSR
- **Auth:** No
- **API:** `GET /api/v1/products/?category={slug}&page={n}&limit=24`
- **NFRs:** SSR for SEO; pagination via URL query param

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Default — Products Loaded | Category has active products |
| 2 | Loading Skeleton | SSR streaming |
| 3 | Empty Category | No products in this category |
| 4 | API Error | Network/server failure |

#### State 1: Default
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Buyer Top Nav]                                                      │
├─────────────────────────────────────────────────────────────────────┤
│ Home > Women's Clothing                                              │  ← breadcrumb
│ Women's Clothing                              Sort: Newest ▼         │  ← h1 + sort
├─────────────────────────────────────────────────────────────────────┤
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│ │[Product] │ │[Product] │ │[Product] │ │[Product] │              │  4-col desktop
│ │Name      │ │Name      │ │Name      │ │Name      │              │
│ │₹999.00   │ │₹1,499.00 │ │₹599.00   │ │₹2,199.00 │              │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘              │
│ [repeat 2 more rows = 12 products on desktop]                       │
├─────────────────────────────────────────────────────────────────────┤
│              [← Prev]  Page 1 of 5  [Next →]                        │  ← pagination
└─────────────────────────────────────────────────────────────────────┘

Mobile (375px): 2-col product grid; same pagination below
```

**Annotations:**
1. 24 products per page; paginated via `?page=N` query param (SSR-compatible)
2. Product card: image (CloudFront, 4:3 ratio), name (2 lines max, ellipsis), price, seller name (1 line)
3. Sort options: Newest (default), Price: Low to High, Price: High to Low, Name A-Z
4. Empty category within a page range → redirect to page 1

#### State 3: Empty Category
```
[Buyer Top Nav]
Home > Women's Clothing
Women's Clothing

        📦
  No products in this category yet.
  Check back soon!

  [  Browse All Products  ]  → /
```

---

## 9. SCR-006: Search Results

### Overview
- **Route:** `/search?q={query}` — SSR
- **Auth:** No
- **API:** `GET /api/v1/products/search/?q={query}&page={n}`
- **NFRs:** PostgreSQL FTS; P95 < 500ms for ≤50K products

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Results Found | FTS returns ≥1 results |
| 2 | Loading Skeleton | SSR in progress |
| 3 | No Results | FTS returns 0 results |
| 4 | Empty Query | User submits blank search |

#### State 1: Results Found
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Buyer Top Nav — search bar pre-filled with query]                  │
├─────────────────────────────────────────────────────────────────────┤
│ Search results for "cotton kurti"                   42 products found│
├─────────────────────────────────────────────────────────────────────┤
│ Sort: Relevance ▼                                                    │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│ │[Product] │ │[Product] │ │[Product] │ │[Product] │              │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘              │
│ [24 per page, 4-col desktop]                                        │
├─────────────────────────────────────────────────────────────────────┤
│              [← Prev]  Page 1 of 2  [Next →]                        │
└─────────────────────────────────────────────────────────────────────┘
```

#### State 3: No Results
```
[Buyer Top Nav — query retained in search bar]

  🔍
  No products found for "xyz123"

  Try:
  · Checking your spelling
  · Using broader keywords

  [  Clear Search  ]     [  Browse All Products  ]
```

#### State 4: Empty Query
```
Redirect to homepage (/) — no results page shown for empty query.
```

---

## 10. SCR-007: Buyer Authentication (Login / Register)

### Overview
- **Route:** `/auth/login` · `/auth/register`
- **Auth:** No — pre-auth screens
- **API:** `POST /api/v1/auth/login/` · `POST /api/v1/auth/register/`
- **NFRs:** JWT RS256; access 24h; refresh 30d; bcrypt cost 12

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Login Tab (Default) | User navigates to /auth/login |
| 2 | Register Tab | User taps "Register" tab |
| 3 | Login — Form Error | Bad credentials |
| 4 | Register — Form Errors | Validation failures |
| 5 | Loading (Submitting) | Form submitted, awaiting API |

#### State 1: Login
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Buyer Top Nav]                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                    ┌────────────────────────┐                       │
│                    │ [Login] [Register]      │  ← tab toggle         │
│                    │─────────────────────── │                       │
│                    │ Email *                 │                       │
│                    │ [_____________________] │                       │
│                    │                         │                       │
│                    │ Password *              │                       │
│                    │ [_____________________] │  ← show/hide toggle   │
│                    │                         │                       │
│                    │ [  Sign In  ]           │  ← orange CTA         │
│                    │                         │                       │
│                    │ Forgot password?        │  ← link (post-MVP)    │
│                    │                         │                       │
│                    │ ─── or ───              │                       │
│                    │ Don't have an account?  │                       │
│                    │ [  Register  ]          │  ← switches to tab 2  │
│                    └────────────────────────┘                       │
└─────────────────────────────────────────────────────────────────────┘
```

#### State 2: Register
```
Same card layout:
│ Full Name *           │
│ [___________________] │
│ Email *               │
│ [___________________] │
│ Password *            │
│ [___________________] │  ← min 8 chars, shown strength indicator
│ Confirm Password *    │
│ [___________________] │
│ Phone (optional)      │
│ [___________________] │
│ [  Create Account  ]  │  ← orange CTA
│                       │
│ Already have account? │
│ [  Sign In  ]         │  ← switches to tab 1
```

**Form Validation:**

| Field | Rule | Error |
|-------|------|-------|
| Email | RFC 5322 format, unique | "Please enter a valid email address" / "Email already registered" |
| Password (register) | ≥8 chars, 1 number | "Password must be at least 8 characters with a number" |
| Confirm Password | Matches password | "Passwords do not match" |
| Phone | 10 digits or empty | "Enter a valid 10-digit Indian mobile number" |

#### State 3: Login — Bad Credentials
```
Form error banner above form:
❌ "Email or password is incorrect. Please try again."
```
Note: Always show generic message — never reveal which field is wrong (security).

---

## 11. SCR-008: Order Tracking (Guest + Authenticated)

### Overview
- **Route:** `/orders/track/{token}` (guest HMAC token) or `/orders/{id}` (authenticated)
- **Auth:** Token-based for guest (HMAC-SHA256, ADR-004); JWT for authenticated
- **API:** `GET /api/v1/orders/track/{token}` · `POST /api/v1/orders/{id}/cancel`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Order Details — Active | Order in progress (any active status) |
| 2 | Loading Skeleton | API fetch in progress |
| 3 | Shipped with AWB | status = SHIPPED; AWB present |
| 4 | Delivered | status = DELIVERED |
| 5 | Cancellable (Pre-Confirmation) | status = PAYMENT_CONFIRMED |
| 6 | Cancellation Blocked | status = PROCESSING or later |
| 7 | Cancelled | status = CANCELLED |
| 8 | Invalid Token | HMAC verification fails |

#### State 1: Order Details
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Buyer Top Nav]                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                   Order #SN-2026-001234                              │
│  Placed 14 May 2026                                                  │
│                                                                      │
│  ──────── Status Timeline ────────────────────────────────────────  │
│  ✅ Payment Confirmed  →  ⏳ Processing  →  ○ Shipped  →  ○ Delivered│
│  (green = done, amber = current, grey = future)                     │
│                                                                      │
│  ──────── Order Items ─────────────────────────────────────────────  │
│  [img 80px]  Blue Cotton Kurti                  ₹999.00             │
│              Qty: 1 · Sold by Priya's Boutique                     │
│                                                                      │
│  ──────── Delivery Address ────────────────────────────────────────  │
│  Anjali Sharma · 45 Park Street, Chennai, TN 600001                │
│                                                                      │
│  ──────── Order Summary ───────────────────────────────────────────  │
│  Total Paid: ₹999.00                                               │
│                                                                      │
│  [  Cancel Order  ]  ← shown if cancellable                        │
└─────────────────────────────────────────────────────────────────────┘
```

**Status timeline colors:** Payment Confirmed = green · Processing = amber · Shipped / Delivered = grey until reached.

#### State 3: Shipped with AWB
```
Status timeline: ✅ Payment Confirmed → ✅ Processing → ✅ Shipped → ○ Delivered

Shipping Info:
📦 Shipped via Delhivery
AWB: 1234567890            [Copy AWB]
Expected delivery: 18 May 2026

Cancel Order button: hidden (order shipped)
```

#### State 5: Cancellable (Pre-Confirmation)
```
[Cancel Order] button visible — orange outline button
On tap → Confirmation modal:
  ┌──────────────────────────────────────┐
  │ Cancel Order?                         │
  │ You will receive a full refund.      │
  │ Refunds appear in 3–7 business days. │
  │                                      │
  │ [Cancel Order]    [Keep Order]       │
  └──────────────────────────────────────┘
```

#### State 6: Cancellation Blocked
```
"Cancel Order" button replaced with:
ℹ️ "Cancellation not available — seller is processing your order." (BR-002)
```

#### State 8: Invalid / Expired Token
```
[Buyer Top Nav]

  🔒
  This tracking link is invalid or expired.

  If you have an account, sign in to view your orders.

  [  Sign In  ]    [  Back to Home  ]
```

---

## 12. SCR-009: Buyer Order History

### Overview
- **Route:** `/orders` — CSR, auth-gated (Buyer)
- **Auth:** Yes — JWT required
- **API:** `GET /api/v1/orders/?buyer=me&page={n}`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Orders List | Buyer has ≥1 order |
| 2 | Loading Skeleton | API fetch |
| 3 | Empty (No Orders) | First-time buyer |
| 4 | API Error | Network failure |

#### State 1: Order List
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Buyer Top Nav]                                                      │
├─────────────────────────────────────────────────────────────────────┤
│ My Orders                                                            │  h1
├─────────────────────────────────────────────────────────────────────┤
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │ Order #SN-2026-001234          Placed 14 May 2026               │ │
│ │ [img 64px]  Blue Cotton Kurti (×1)              ₹999.00         │ │
│ │             ✅ Delivered                                          │ │
│ │                                          [Track Order] →         │ │
│ └─────────────────────────────────────────────────────────────────┘ │
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │ Order #SN-2026-001210          Placed 10 May 2026               │ │
│ │ [img 64px]  Red Silk Dupatta (×2)               ₹1,198.00       │ │
│ │             ⏳ Processing                                         │ │
│ │                                          [Track Order] →         │ │
│ └─────────────────────────────────────────────────────────────────┘ │
│              [← Prev]  Page 1 of 1  [Next →]                        │
└─────────────────────────────────────────────────────────────────────┘
```

#### State 3: No Orders
```
        📦
  You haven't placed any orders yet.

  [  Start Shopping  ]  → /
```

---

## 13. SCR-010: Seller Registration

### Overview
- **Route:** `/seller/register` — CSR (no auth shell — pre-registration)
- **Auth:** No
- **API:** `POST /api/v1/sellers/register/`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Registration Form (Default) | User opens /seller/register |
| 2 | Loading (Submitting) | Form submitted |
| 3 | Form Validation Errors | Validation failures |
| 4 | Submission Success | API returns 201 |
| 5 | Email Already Registered | Duplicate email |

#### State 1: Registration Form
```
Desktop (1440px) — centered card, max-width 560px:
┌─────────────────────────────────────────────────────────────────────┐
│ ShopNest Logo (centered top)                                         │
├─────────────────────────────────────────────────────────────────────┤
│                ┌──────────────────────────────────┐                 │
│                │ Register as a Seller              │  h1             │
│                │ Start selling on ShopNest today   │  subtitle       │
│                │──────────────────────────────── ─│                 │
│                │ Full Name *                       │                 │
│                │ [________________________________]│                 │
│                │ Email *                           │                 │
│                │ [________________________________]│                 │
│                │ Password *                        │                 │
│                │ [________________________________]│  ← strength bar │
│                │ Business Name *                   │                 │
│                │ [________________________________]│                 │
│                │ Phone *                           │                 │
│                │ [+91][__________________________]│                 │
│                │ City *                            │                 │
│                │ [________________________________]│                 │
│                │ GSTIN (optional)                  │                 │
│                │ [________________________________]│  ← 15-char hint │
│                │                                   │                 │
│                │ [   Create Seller Account   ]     │  ← orange CTA   │
│                │                                   │                 │
│                │ Already selling? Sign in →         │                 │
│                └──────────────────────────────────┘                 │
└─────────────────────────────────────────────────────────────────────┘
```

**Form Validation:**

| Field | Rule | Error |
|-------|------|-------|
| Full Name | Required, 2–100 chars | "Please enter your full name" |
| Email | RFC 5322, unique | "Please enter a valid email" / "Email already registered" |
| Password | ≥8 chars, 1 number | "Password must be ≥8 chars with a number" |
| Business Name | Required, 2–200 chars | "Please enter your business name" |
| Phone | 10 digits starting 6-9 | "Enter a valid 10-digit Indian mobile number" |
| City | Required | "Please enter your city" |
| GSTIN | Optional; 15-char alphanumeric if provided | "GSTIN must be 15 characters" |

#### State 4: Submission Success (Pending Review)
```
✅ Registration submitted!

Your seller account is under review.
You'll receive an email at priya@example.com
once our team approves your application.

This usually takes 1–2 business days.

[  Continue to Store Setup  ]  ← → SCR-011
```

---

## 14. SCR-011: Store Setup Wizard

### Overview
- **Route:** `/seller/setup` — CSR, auth-gated (Seller role, any subscription status)
- **Auth:** Yes (Seller)
- **API:** `GET /api/v1/seller/profile/` · `PATCH /api/v1/seller/store/`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Step 1 — Store Info | Wizard starts |
| 2 | Step 2 — Categories | After Step 1 saved |
| 3 | Step 3 — Review & Confirm | All fields filled |
| 4 | Upload in Progress | Image being uploaded to S3 |
| 5 | Image Error | File too large or wrong type |
| 6 | Wizard Complete | Step 3 submitted |

#### State 1: Step 1 — Store Info
```
Desktop (1440px) — centered card, max-width 640px:
┌─────────────────────────────────────────────────────────────────────┐
│ [Seller top bar — minimal (no sidebar yet)]                          │
├─────────────────────────────────────────────────────────────────────┤
│          Set Up Your Store                                           │  h1
│          ● ○ ○  Step 1 of 3: Store Information                      │  progress
│                                                                      │
│  Store Name *                                                        │
│  [____________________________________________]                      │
│                                                                      │
│  Store Logo                                                          │
│  ┌──────────────────────────────────────────┐                       │
│  │  📷  Drag & drop or click to upload      │                       │
│  │      JPG · PNG · Max 2MB                 │                       │
│  └──────────────────────────────────────────┘                       │
│  [thumbnail preview after upload]                                    │
│                                                                      │
│  Store Banner                                                        │
│  ┌──────────────────────────────────────────┐                       │
│  │  📷  Drag & drop or click to upload      │                       │
│  │      JPG · PNG · Max 5MB                 │                       │
│  └──────────────────────────────────────────┘                       │
│                                                                      │
│  Store Description (optional)                                        │
│  [______________________________]  (3 rows textarea)                │
│                                                                      │
│  [  Next: Choose Categories  ]  ← orange CTA                       │
└─────────────────────────────────────────────────────────────────────┘
```

#### State 2: Step 2 — Categories
```
│          Set Up Your Store                                           │
│          ● ● ○  Step 2 of 3: Store Categories                      │
│                                                                      │
│  Choose categories that best describe your products.                │
│  Select up to 3.                                                     │
│                                                                      │
│  [✓ Women's Clothing]  [  Men's Clothing  ]  [  Electronics  ]     │  ← multi-select chips
│  [  Home & Kitchen  ]  [  Sports          ]  [  Books         ]     │
│                                                                      │
│  [← Back]              [  Next: Review  ]                           │
```

#### State 3: Step 3 — Review & Confirm
```
│          Set Up Your Store                                           │
│          ● ● ●  Step 3 of 3: Review                                │
│                                                                      │
│  Store Name:    Priya's Boutique                                    │
│  Logo:          [small preview]                                     │
│  Banner:        [small preview]                                     │
│  Categories:    Women's Clothing, Home & Kitchen                    │
│                                                                      │
│  [← Back]       [  Launch Store  ]  ← orange CTA → SCR-012        │
```

---

## 15. SCR-012: Seller Subscription Signup

### Overview
- **Route:** `/seller/subscribe` — CSR, auth-gated (Seller role)
- **Auth:** Yes (Seller)
- **API:** `POST /api/v1/payments/subscriptions/create/`
- **Payment:** Razorpay Subscriptions API — ShopNest opens modal, never designs payment UI

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Plan Display (Default) | Seller reaches subscription step |
| 2 | Razorpay Modal Active | Seller clicks "Subscribe" |
| 3 | Subscription Success | Razorpay webhook confirms |
| 4 | Payment Failed | Razorpay returns failure |

#### State 1: Plan Display
```
Desktop (1440px) — centered card, max-width 560px:
┌─────────────────────────────────────────────────────────────────────┐
│ [Seller top bar — minimal]                                           │
├─────────────────────────────────────────────────────────────────────┤
│              Activate Your Store                                     │  h1
│              Subscribe to start listing products                    │  subtitle
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  ShopNest Seller Plan                                         │  │
│  │  ₹1,999 / month                                              │  │
│  │                                                               │  │
│  │  ✓  Unlimited product listings                               │  │
│  │  ✓  Admin approval for each product                          │  │
│  │  ✓  Weekly payouts via Razorpay                              │  │
│  │  ✓  Email order notifications                                │  │
│  │  ✓  Seller dashboard and analytics                           │  │
│  │                                                               │  │
│  │  [   Subscribe for ₹1,999/month   ]  ← orange CTA           │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  🔒 Secure payment powered by Razorpay                             │
│  You can cancel anytime from your dashboard.                        │
└─────────────────────────────────────────────────────────────────────┘
```

#### State 2: Razorpay Modal Active
```
ShopNest page dimmed (opacity-30, pointer-events-none)
Razorpay Checkout.js modal on top (role="dialog")
Annotation: "Razorpay Subscription modal — ShopNest never designs or sees payment data"
```

#### State 3: Subscription Success
```
        ✅

  You're all set!
  Your store is now active.

  Start listing your products to reach buyers.

  [  Go to Dashboard  ]   → SCR-013
  [  Add First Product ]  → SCR-003
```

#### State 4: Payment Failed
```
        ❌

  Payment Failed

  No amount was charged.
  Your store will be activated once payment succeeds.

  [  Try Again  ]  ← re-opens Razorpay modal
```

---

## 16. SCR-013: Seller Dashboard

### Overview
- **Route:** `/seller/dashboard` — CSR, auth-gated (Seller + Active subscription)
- **Auth:** Yes (Seller)
- **API:** `GET /api/v1/seller/dashboard/` (aggregated: orders, products, payouts)

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Dashboard Default (Active) | Active seller with data |
| 2 | Loading Skeleton | API fetch |
| 3 | Empty Dashboard | New seller, no data yet |
| 4 | Subscription Inactive | Subscription lapsed |

#### State 1: Default Dashboard
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Seller top header: ShopNest Seller · Priya's Boutique · ● Active]  │
├──────────────────┬──────────────────────────────────────────────────┤
│ [Seller Sidebar] │ Dashboard                                        h1│
│ ● Dashboard      │                                                    │
│   Products       │  ┌──────────┐ ┌──────────┐ ┌──────────┐         │
│   Orders         │  │  Orders  │ │ Revenue  │ │ Products │         │
│   Payouts        │  │    12    │ │₹24,500   │ │    8     │         │
│   ─────────      │  │  this mo.│ │  this mo.│ │  active  │         │
│   Settings       │  └──────────┘ └──────────┘ └──────────┘         │
│   Subscription ✓ │                                                    │
│                  │  Recent Orders                        [View All →]│
│                  │  ┌──────────────────────────────────────────────┐│
│                  │  │ #SN-001234  Kurti ×1  ₹999  ⏳ Processing   ││
│                  │  │ #SN-001210  Dupatta×2 ₹1,198 ✅ Delivered   ││
│                  │  └──────────────────────────────────────────────┘│
│                  │                                                    │
│                  │  Quick Actions                                    │
│                  │  [+ Add Product]  [View Orders]  [View Payouts]  │
└──────────────────┴──────────────────────────────────────────────────┘
```

**KPI Cards:**
- Orders This Month: count of orders with `created_at` in current month
- Revenue This Month: sum of `gross_amount_paise` for this month ÷ 100 (₹ format)
- Active Products: count of products with status = ACTIVE

#### State 3: Empty Dashboard
```
Same sidebar layout.
Main area:
        🏪
  Welcome to ShopNest, Priya!
  Your store is ready. Add products to start selling.

  [  + Add Your First Product  ]  ← orange CTA
```

---

## 17. SCR-014: Seller Order Management

### Overview
- **Route:** `/seller/orders` — CSR, auth-gated (Seller)
- **Auth:** Yes (Seller)
- **API:** `GET /api/v1/seller/orders/` · `POST /api/v1/seller/orders/{id}/confirm/` · `POST /api/v1/seller/orders/{id}/ship/` · `POST /api/v1/seller/orders/{id}/deliver/`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Order List (Default) | Active seller, has orders |
| 2 | Loading Skeleton | API fetch |
| 3 | Empty (No Orders) | No orders yet |
| 4 | API Error | Network failure |
| 5 | Order Expanded — Awaiting Confirmation | status = PAYMENT_CONFIRMED |
| 6 | Order Expanded — Awaiting Shipment | status = PROCESSING |
| 7 | Order Expanded — Shipped | status = SHIPPED |

#### State 1: Order List
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Seller header]                                                      │
├──────────────────┬──────────────────────────────────────────────────┤
│ [Seller Sidebar] │  Orders                                       h1  │
│   Orders ●       │                                                    │
│                  │  [All ▼]  [Status ▼]          [Search orders...] │
│                  │                                                    │
│                  │  ┌─────────────────────────────────────────────┐ │
│                  │  │ORDER  │ BUYER          │ ITEMS │ TOTAL │STATUS│ │
│                  │  ├─────────────────────────────────────────────┤ │
│                  │  │#001234│ Anjali S.      │ 1     │₹999  │⏳ Pay.│ │
│                  │  │#001210│ Rahul K.       │ 2     │₹1,198│✅ Del.│ │
│                  │  │#001199│ Meena R.       │ 1     │₹2,499│📦 Ship│ │
│                  │  └─────────────────────────────────────────────┘ │
└──────────────────┴──────────────────────────────────────────────────┘
```

**Buyer privacy:** Shows first name + last initial only (e.g., "Anjali S.").

#### State 5: Order Expanded — Awaiting Confirmation
```
Expanded row (or side panel):
Order #SN-001234
Status: ⏳ Payment Confirmed
Placed: 14 May 2026

Items:
  Blue Cotton Kurti × 1       ₹999.00

Delivery:
  Anjali Sharma
  45 Park Street, Chennai, TN 600001

[  Confirm Order  ]  ← orange CTA
Confirming locks out buyer from cancellation (BR-002)
```

#### State 6: Order Expanded — Awaiting Shipment
```
Status: 🔄 Processing

Shipping Details:
Courier Name *   [_________________________]
AWB Number *     [_________________________]

[  Mark as Shipped  ]  ← orange CTA
Validation: both fields required before submit
```

#### State 7: Order Expanded — Shipped
```
Status: 📦 Shipped
Courier: Delhivery
AWB: 1234567890

[  Mark as Delivered  ]  ← secondary CTA (optional)
```

---

## 18. SCR-015: Seller Payouts

### Overview
- **Route:** `/seller/payouts` — CSR, auth-gated (Seller)
- **Auth:** Yes (Seller)
- **API:** `GET /api/v1/seller/payouts/` · `GET /api/v1/seller/payouts/{id}/`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Payout History List | Seller has ≥1 payout |
| 2 | Loading Skeleton | API fetch |
| 3 | Empty (No Payouts Yet) | No payouts processed |
| 4 | Payout Detail | Seller clicks payout row |

#### State 1: Payout History
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Seller header]                                                      │
├──────────────────┬──────────────────────────────────────────────────┤
│ [Seller Sidebar] │  Payouts                                      h1  │
│   Payouts ●      │                                                    │
│                  │  Next payout: Monday 19 May 2026                  │
│                  │  Pending amount: ₹8,500.00                       │
│                  │                                                    │
│                  │  Payout History                                    │
│                  │  ┌──────────────────────────────────────────────┐ │
│                  │  │DATE        │ GROSS    │ FEE   │ NET    │STATUS│ │
│                  │  ├──────────────────────────────────────────────┤ │
│                  │  │12 May 2026 │ ₹24,500 │ ₹490  │₹24,010│ Paid │ │
│                  │  │5 May 2026  │ ₹18,200 │ ₹364  │₹17,836│ Paid │ │
│                  │  └──────────────────────────────────────────────┘ │
└──────────────────┴──────────────────────────────────────────────────┘
```

**Column definitions:**
- Gross: sum of order amounts in payout cycle
- Fee: Razorpay gateway fee (~2% — ShopNest absorbs, shown for transparency)
- Net: gross - fee (actual bank deposit amount)
- Status: Paid / Pending / Failed

#### State 3: No Payouts
```
        💰
  No payouts yet.
  Your first payout will be processed after your first delivered order.

  Payouts are processed every Monday.
```

---

## 19. SCR-016: Seller Subscription Management

### Overview
- **Route:** `/seller/subscription` — CSR, auth-gated (Seller)
- **Auth:** Yes (Seller)
- **API:** `GET /api/v1/seller/subscription/` · `POST /api/v1/payments/subscriptions/cancel/`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Active Subscription | subscription.status = ACTIVE |
| 2 | Expired / Lapsed | subscription.status = EXPIRED |
| 3 | Cancellation Confirm Modal | Seller clicks Cancel |
| 4 | Cancelled | subscription.status = CANCELLED |

#### State 1: Active Subscription
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Seller header]                                                      │
├──────────────────┬──────────────────────────────────────────────────┤
│ [Seller Sidebar] │  Subscription                                 h1  │
│   Subscription ✓ │                                                    │
│                  │  ┌──────────────────────────────────────────────┐ │
│                  │  │ ● Active Subscription                         │ │
│                  │  │ Plan: ShopNest Seller Plan                   │ │
│                  │  │ ₹1,999 / month                              │ │
│                  │  │ Next billing: 14 June 2026                   │ │
│                  │  │ Payment method: Razorpay (UPI)               │ │
│                  │  │                                               │ │
│                  │  │ [  Cancel Subscription  ]  ← gray outline    │ │
│                  │  └──────────────────────────────────────────────┘ │
│                  │                                                    │
│                  │  Billing History                                   │
│                  │  ┌──────────────────────────────────────────────┐ │
│                  │  │DATE        │ AMOUNT   │ STATUS │ INVOICE     │ │
│                  │  │14 May 2026 │ ₹1,999  │ Paid   │ Download PDF│ │
│                  │  │14 Apr 2026 │ ₹1,999  │ Paid   │ Download PDF│ │
│                  │  └──────────────────────────────────────────────┘ │
└──────────────────┴──────────────────────────────────────────────────┘
```

#### State 2: Expired
```
⚠️ Subscription Expired

Your store is currently hidden from buyers.
Renew to reactivate.

[  Renew for ₹1,999/month  ]  ← orange CTA → Razorpay modal
```

#### State 3: Cancellation Confirm Modal
```
┌──────────────────────────────────────────────────────┐
│ Cancel Subscription?                                  │
│                                                       │
│ Your store will be deactivated at end of billing     │
│ period (14 June 2026). You can renew at any time.   │
│                                                       │
│ [  Keep Subscription  ]    [  Yes, Cancel  ]          │
└──────────────────────────────────────────────────────┘
```

---

## 20. SCR-017: Seller Store Settings

### Overview
- **Route:** `/seller/settings` — CSR, auth-gated (Seller)
- **Auth:** Yes (Seller)
- **API:** `GET /api/v1/seller/store/` · `PATCH /api/v1/seller/store/` · `PATCH /api/v1/seller/payout-details/`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Settings Form (Default) | Seller opens settings |
| 2 | Save Loading | PATCH in progress |
| 3 | Save Success | PATCH 200 |
| 4 | Save Error | PATCH failure |

#### State 1: Settings Form
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Seller header]                                                      │
├──────────────────┬──────────────────────────────────────────────────┤
│ [Seller Sidebar] │  Store Settings                               h1  │
│   Settings ●     │                                                    │
│                  │  Store Information                                 │
│                  │  Store Name *  [Priya's Boutique_____________]    │
│                  │  City *        [Bangalore____________________]    │
│                  │  Description   [_________________________ 3 rows] │
│                  │  Logo          [current logo thumb] [Change]      │
│                  │  Banner        [current banner thumb] [Change]    │
│                  │                                                    │
│                  │  ─────────────────────────────────────────────   │
│                  │  Payout Details                                    │
│                  │  Bank Account Number *                            │
│                  │  [●●●●●●●●●●●● 4321]  [Edit]  ← masked display  │
│                  │  IFSC Code *   [HDFC0001234___________________]   │
│                  │  Account Name *[Priya Sharma__________________]   │
│                  │                                                    │
│                  │  [  Save Changes  ]  ← orange CTA               │
└──────────────────┴──────────────────────────────────────────────────┘
```

**Security note:** Bank account number displayed masked (last 4 digits only). "Edit" button reveals input field — masked on save. AES-256-GCM encrypted at rest (SECURITY-ARCHITECTURE.md).

---

## 21. SCR-018: Admin Login

### Overview
- **Route:** `/admin/login` — CSR (no shell — pre-auth)
- **Auth:** No
- **API:** `POST /api/v1/auth/admin/login/`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Login Form | Admin opens /admin/login |
| 2 | Submitting | Form submitted |
| 3 | Login Error | Bad credentials |
| 4 | Success | Valid admin credentials → /admin/dashboard |

#### State 1: Login Form
```
Desktop (centered card, max-width 400px):
┌─────────────────────────────────────────────────────────────────────┐
│ ShopNest Logo (centered)                                             │
│ Admin Portal                                                         │
├─────────────────────────────────────────────────────────────────────┤
│              ┌────────────────────────────┐                         │
│              │ Admin Sign In              │  h2                      │
│              │                            │                         │
│              │ Email *                    │                         │
│              │ [________________________] │                         │
│              │                            │                         │
│              │ Password *                 │                         │
│              │ [________________________] │  ← show/hide             │
│              │                            │                         │
│              │ [  Sign In  ]              │  ← orange CTA            │
│              └────────────────────────────┘                         │
└─────────────────────────────────────────────────────────────────────┘
```

#### State 3: Login Error
```
Red error banner above form:
❌ "Invalid email or password."
```

---

## 22. SCR-019: Admin Seller Approval Queue

### Overview
- **Route:** `/admin/sellers` — CSR, auth-gated (Admin role)
- **Auth:** Yes (Admin)
- **API:** `GET /api/v1/admin/sellers/?status=PENDING_REVIEW` · `POST /api/v1/admin/sellers/{id}/approve/` · `POST /api/v1/admin/sellers/{id}/reject/`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Queue Default (Pending Tab) | Admin opens seller queue |
| 2 | Loading Skeleton | API fetch |
| 3 | Empty Queue | No pending sellers |
| 4 | Seller Row Expanded | Admin clicks a seller row |
| 5 | Approve Confirm | Admin clicks Approve |
| 6 | Reject Modal | Admin clicks Reject |
| 7 | API Error | Action fails |

#### State 1: Queue Default
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Admin header: ShopNest Admin]                                       │
├──────────────────┬──────────────────────────────────────────────────┤
│ [Admin Sidebar]  │  Seller Applications                          h1  │
│ ● Seller Queue   │                                                    │
│                  │  [Pending (3)] [Approved] [Rejected]              │  ← tab bar
│                  │                                                    │
│                  │  ┌──────────────────────────────────────────────┐ │
│                  │  │ NAME         │ BUSINESS         │ CITY │ DATE │ │
│                  │  ├──────────────────────────────────────────────┤ │
│                  │  │ Priya Sharma │ Priya's Boutique  │ BLR  │May14│ │
│                  │  │ Vikram Nair  │ Vikram Handicrafts│ CHN  │May13│ │
│                  │  │ Sunita Devi  │ Sunita Sarees     │ DEL  │May12│ │
│                  │  └──────────────────────────────────────────────┘ │
└──────────────────┴──────────────────────────────────────────────────┘
```

#### State 4: Seller Row Expanded
```
Expanded row shows additional details:
  Email:    priya@example.com
  Phone:    +91 98765 43210
  GSTIN:    29ABCDE1234F1Z5 (if provided)
  Registered: 14 May 2026

  [  Approve  ]  ← green button      [  Reject  ]  ← red outline button
```

#### State 6: Reject Modal
```
┌──────────────────────────────────────────────┐
│ Reject Seller Application                     │
│                                               │
│ Rejection reason *                            │
│ ┌────────────────────────────────────────┐   │
│ │ Please describe why this application   │   │
│ │ is being rejected...                   │   │
│ └────────────────────────────────────────┘   │
│ This reason will be emailed to the seller.   │
│                                               │
│ [  Cancel  ]         [  Confirm Reject  ]     │
└──────────────────────────────────────────────┘
```

---

## 23. SCR-020: Admin Product Approval Queue

### Overview
- **Route:** `/admin/products` — CSR, auth-gated (Admin role)
- **Auth:** Yes (Admin)
- **API:** `GET /api/v1/admin/products/?status=PENDING_REVIEW` · `POST /api/v1/admin/products/{id}/approve/` · `POST /api/v1/admin/products/{id}/reject/`

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Queue Default (Pending Tab) | Admin opens product queue |
| 2 | Loading Skeleton | API fetch |
| 3 | Empty Queue | No pending products |
| 4 | Product Row Expanded | Admin clicks product row |
| 5 | Image Enlarged | Admin clicks product image |
| 6 | Approve Action | Admin approves |
| 7 | Reject Modal | Admin rejects with reason |

#### State 1: Queue Default
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Admin header]                                                       │
├──────────────────┬──────────────────────────────────────────────────┤
│ [Admin Sidebar]  │  Product Approvals                            h1  │
│   Product Queue● │                                                    │
│                  │  [Pending (7)] [Approved] [Rejected]              │
│                  │                                                    │
│                  │  ┌──────────────────────────────────────────────┐ │
│                  │  │[Img]│ PRODUCT NAME         │ SELLER   │ PRICE │ │
│                  │  ├──────────────────────────────────────────────┤ │
│                  │  │[48] │ Blue Cotton Kurti     │ Priya B. │₹999  │ │
│                  │  │[48] │ Green Embr. Saree     │ Priya B. │₹2,499│ │
│                  │  │[48] │ Red Silk Dupatta      │ Vikram H.│₹599  │ │
│                  │  └──────────────────────────────────────────────┘ │
└──────────────────┴──────────────────────────────────────────────────┘
```

#### State 4: Product Row Expanded
```
Full product details panel:
  [Image 1] [Image 2] [Image 3]  ← clickable to enlarge (State 5)
  Name: Blue Cotton Kurti
  Description: Handloom cotton kurti, perfect for daily wear. Available in sizes S, M, L, XL.
  Price: ₹999.00
  Stock: 50 units
  Category: Women's Clothing
  Seller: Priya's Boutique (Bangalore) · priya@example.com

  [  Approve  ]   [  Reject  ]
```

#### State 7: Reject Modal
```
┌──────────────────────────────────────────────┐
│ Reject Product Listing                        │
│                                               │
│ Rejection reason *                            │
│ ┌────────────────────────────────────────┐   │
│ │ E.g. "Description is too short"        │   │
│ └────────────────────────────────────────┘   │
│ This reason will be shown in seller's        │
│ product dashboard and emailed to them.       │
│                                               │
│ [  Cancel  ]         [  Confirm Reject  ]     │
└──────────────────────────────────────────────┘
```

---

## 24. SCR-021: Admin Dashboard

### Overview
- **Route:** `/admin/dashboard` — CSR, auth-gated (Admin role)
- **Auth:** Yes (Admin)
- **API:** `GET /api/v1/admin/dashboard/` (aggregated stats)

### States

| # | State | Trigger |
|---|-------|---------|
| 1 | Dashboard Default | Admin logs in |
| 2 | Loading Skeleton | API fetch |

#### State 1: Admin Dashboard Default
```
Desktop (1440px):
┌─────────────────────────────────────────────────────────────────────┐
│ [Admin header]                                                       │
├──────────────────┬──────────────────────────────────────────────────┤
│ [Admin Sidebar]  │  Admin Dashboard                              h1  │
│ ● Dashboard      │                                                    │
│                  │  Platform Health                                   │
│                  │  ┌──────────┐ ┌──────────┐ ┌──────────┐         │
│                  │  │Sellers   │ │ Products │ │  Orders  │         │
│                  │  │Active:12 │ │Active: 48│ │Today: 34 │         │
│                  │  │Pending: 3│ │Pending: 7│ │Month: 412│         │
│                  │  └──────────┘ └──────────┘ └──────────┘         │
│                  │                                                    │
│                  │  Action Required                                   │
│                  │  ┌──────────────────────────────────────────────┐ │
│                  │  │ 3 seller applications pending review          │ │
│                  │  │ [  Review Sellers  ]  →  /admin/sellers       │ │
│                  │  ├──────────────────────────────────────────────┤ │
│                  │  │ 7 products pending approval                   │ │
│                  │  │ [  Review Products  ]  →  /admin/products     │ │
│                  │  └──────────────────────────────────────────────┘ │
└──────────────────┴──────────────────────────────────────────────────┘
```

**KPI Cards:**
- Active Sellers: sellers with status = ACTIVE and subscription = ACTIVE
- Pending Sellers: sellers with status = PENDING_REVIEW
- Active Products: products with status = ACTIVE
- Pending Products: products with status = PENDING_REVIEW
- Orders Today: count with `created_at` = today
- Orders This Month: count with `created_at` in current calendar month

---

## 25. Cognitive Walkthrough Summary (All Journeys)

> Walkthroughs performed per Wharton et al. (1994). Each journey traced for primary persona.
> Full walkthroughs for SCR-001, SCR-002, SCR-003 documented above (§4–6).

| Journey | Result | Critical Issues | Resolution |
|---------|--------|----------------|------------|
| Rahul: Homepage → PDP → Cart → Checkout → Success | ✅ Pass | Cart toast may be missed on mobile | Cart badge provides persistent confirmation |
| Anjali: WhatsApp link → PDP → Guest Checkout → Success | ✅ Pass | Tracking URL may be missed | Copy button + email delivery on success page |
| Anjali: Order Tracking → Cancel | ✅ Pass | Cancellation state is ambiguous | Clear "Cancellation not available" message post-confirmation |
| Rahul: Search → Results → PDP | ✅ Pass | None | — |
| Priya: Registration → Setup → Subscribe → Dashboard | ✅ Pass | Logo/banner upload on mobile may be confusing | File size/type shown before upload; progress bar |
| Priya: Add Product → Pending Review | ✅ Pass | Description min length may be missed | Counter turns red; submit disabled until met |
| Priya: Confirm → Ship Order | ✅ Pass | AWB field requirement not obvious | Inline error if shipped without AWB |
| Admin: Seller Approval | ✅ Pass | Rejection reason field required | Modal enforces field before confirm |
| Admin: Product Approval | ✅ Pass | Image quality judgement from small thumbnail | Images expandable full-size on click |

---

## 26. Cross-Screen Consistency Checklist

| Rule | Applies To | Verified |
|------|-----------|---------|
| Primary CTA: orange (#F97316), 12px vertical padding, rounded-xl | All screens | ✅ |
| Error state: red border + red text below field, fires on blur | All forms | ✅ |
| Success state: green border + green checkmark, on blur | All forms | ✅ |
| Status badge: [dot][label], text-xs font-semibold | SCR-001, 003, 008, 009, 014, 019, 020 | ✅ |
| Skeleton loader: shimmer gradient, matches element dimensions | All loading states | ✅ |
| "Pending Review" term (not "Pending Approval") | SCR-003, 008, 013, 019, 020 | ✅ |
| ₹ prefix, 2 decimal places, Indian thousand separator | All price displays | ✅ |
| Razorpay modal: ShopNest page dimmed (opacity-30); modal annotated as Razorpay-owned | SCR-002, SCR-012 | ✅ |
| focus-visible ring: 2px solid #1A56DB, outline-offset: 2px | All interactive elements | ✅ |
| role="alert" aria-live="assertive" on error banners | All error panels | ✅ |
| Triple-Gate check documented: Seller=ACTIVE + Sub=ACTIVE + Product=ACTIVE | SCR-003, SCR-004, SCR-005, SCR-006 | ✅ |
