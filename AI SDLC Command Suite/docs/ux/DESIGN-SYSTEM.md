# DESIGN-SYSTEM.md — ShopNest Design System

**Phase:** 05 — UX Design (Full Production Run)
**Date:** 2026-05-15
**Scope:** All 21 screens (SCR-001 through SCR-021)
**Wireframe Reference Frame:** 1440px desktop (HTML files use `min-width: 1280px` viewport lock). Mobile-first remains the implementation target; wireframes show desktop layout to enable full component inspection without phone-frame clipping.

---

## 1. Design Principles

| Principle | Application |
|-----------|------------|
| **Mobile-first implementation** | All components designed for 375px mobile first; wireframe HTML files use 1440px reference frame to enable full component inspection. Desktop enhancements are additive — never remove mobile-designed functionality. |
| **Clarity over cleverness** | Straightforward copy ("Pending Review", "Out of Stock") — no jargon |
| **Status always visible** | Product and seller status badges are always on-screen; no buried metadata |
| **Inline feedback** | Validation errors fire on blur; success confirmed with inline checkmarks |
| **India context** | ₹ currency symbol; 10-digit Indian mobile format; Razorpay brand trust |

---

## 2. Color Tokens

### Primary Palette

| Token | Hex | Tailwind Class | Usage |
|-------|-----|---------------|-------|
| `color-primary` | `#1A56DB` | `blue-700` | Links, active tabs, borders on focus, seller nav active |
| `color-primary-light` | `#EBF5FF` | `blue-50` | Address highlight boxes, info banners background |
| `color-cta` | `#F97316` | `orange-500` | Primary CTA buttons (Add to Cart, Buy Now, Pay, Submit) |
| `color-cta-hover` | `#EA6C0A` | `orange-600` | CTA button hover state |

### Status Palette

| Token | Hex | Tailwind | Usage |
|-------|-----|---------|-------|
| `color-success` | `#16A34A` | `green-600` | Active status dot, In Stock badge, form field valid |
| `color-success-bg` | `#F0FDF4` | `green-50` | Valid field background |
| `color-warning` | `#D97706` | `amber-600` | Pending Review dot, suspension banner icon |
| `color-warning-bg` | `#FFFBEB` | `amber-50` | Suspension banner background |
| `color-error` | `#DC2626` | `red-600` | Rejected dot, Out of Stock badge, form error text |
| `color-error-bg` | `#FEF2F2` | `red-50` | Error field background |
| `color-disabled` | `#6B7280` | `gray-500` | Disabled buttons, hidden product labels |
| `color-disabled-bg` | `#F9FAFB` | `gray-50` | Disabled field background |

### Neutral Palette

| Token | Hex | Tailwind | Usage |
|-------|-----|---------|-------|
| `color-text-primary` | `#111827` | `gray-900` | Headings, prices, product names |
| `color-text-secondary` | `#6B7280` | `gray-500` | Meta text, labels, subtitles |
| `color-text-muted` | `#9CA3AF` | `gray-400` | Placeholder text, secondary labels |
| `color-border` | `#E5E7EB` | `gray-200` | Default borders, dividers |
| `color-surface` | `#FFFFFF` | `white` | Card backgrounds, modals |
| `color-background` | `#F9FAFB` | `gray-50` | Page background |

---

## 3. Typography

### Type Scale

| Role | Size (mobile) | Size (desktop ≥1024px) | Weight | Line Height | Tailwind |
|------|--------------|----------------------|--------|-------------|---------|
| Page title (h1 — Product Name) | 20px | 24px | 700 | 1.3 | `text-xl font-bold` / `md:text-2xl` |
| Section heading (h2) | 16px | 20px | 700 | 1.4 | `text-base font-bold` / `md:text-xl` |
| Body — primary | 14px | 14px | 400 | 1.5 | `text-sm` |
| Body — secondary | 12px | 12px | 400 | 1.5 | `text-xs` |
| Price (prominent) | 24px | 28px | 700 | 1.2 | `text-2xl font-bold` / `md:text-3xl` |
| Price (list) | 14px | 16px | 700 | 1.4 | `text-sm font-bold` / `md:text-base` |
| Label / caption | 11px | 11px | 600 | 1.4 | `text-xs font-semibold` |
| Button text — primary CTA | 14px | 16px | 700 | 1 | `text-sm font-bold` / `md:text-base` |
| Table header (seller dashboard) | — | 12px | 600 | 1.4 | `text-xs font-semibold uppercase tracking-wide` |

### Font Family
- **Primary:** System font stack — `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`
- Rationale: Renders at native quality on iOS (SF Pro) and Android (Roboto); zero web font load time; critical for P95 LCP < 2.5s on mobile 4G

### Currency Display Rule
- Always prefix with ₹ symbol
- Always show 2 decimal places: `₹999.00`, `₹2,197.00`
- Source value is integer paise → display = `paise / 100` formatted to 2dp
- Thousands separator: Indian format (₹1,00,000 for lakhs) — use JS `toLocaleString('en-IN')`

---

## 4. Spacing System

Base unit: **4px (0.25rem)**

| Token | Size | Tailwind | Usage |
|-------|------|---------|-------|
| `space-1` | 4px | `p-1` / `m-1` | Tight internal padding |
| `space-2` | 8px | `p-2` / `m-2` | Icon gaps, small margins |
| `space-3` | 12px | `p-3` / `m-3` | Card internal padding (compact) |
| `space-4` | 16px | `p-4` / `m-4` | Standard card padding, screen margins |
| `space-6` | 24px | `p-6` / `m-6` | Section spacing |
| `space-8` | 32px | `p-8` / `m-8` | Large vertical gaps |

**Screen edge margin:** 16px (p-4) on mobile · 40px (px-10) on desktop (1024px+).
**Desktop max-width container:** `max-w-screen-xl mx-auto` (1280px) — used on all 3 screens.

---

## 5. Elevation & Shadow

| Level | CSS | Usage |
|-------|-----|-------|
| Level 0 | `none` | Flat cards in list views |
| Level 1 | `0 1px 3px rgba(0,0,0,0.08)` | Subtle separation (nav bar) |
| Level 2 | `0 4px 16px rgba(0,0,0,0.12)` | Modals, bottom sheets, FAB |
| Level 3 | `0 8px 32px rgba(0,0,0,0.18)` | Dropdown menus, popovers |

---

## 6. Border Radius

| Token | Size | Tailwind | Usage |
|-------|------|---------|-------|
| `radius-sm` | 6px | `rounded` | Small badges, chips |
| `radius-md` | 8px | `rounded-lg` | Input fields, product image thumbnails |
| `radius-lg` | 12px | `rounded-xl` | Buttons, cards, modals |
| `radius-full` | 9999px | `rounded-full` | Status dots, avatar circles, FAB |

---

## 7. Component Inventory

### 7.1 Button — Primary CTA

| Property | Value |
|----------|-------|
| Background | `color-cta` (#F97316) |
| Text | White, 14px, 600 weight |
| Border radius | 12px (rounded-xl) |
| Padding | 12px vertical, 16px horizontal |
| Min width | Full width on mobile |
| Hover | Background → `color-cta-hover` |
| Disabled | Background → gray-200, text → gray-400, cursor not-allowed |
| Loading | Spinner replaces text; button disabled |

**Usage:** Add to Cart (secondary style), Buy Now, Pay ₹{amount}, Submit for Review, Proceed to Checkout, Renew Subscription

### 7.2 Button — Secondary (Outline)

| Property | Value |
|----------|-------|
| Background | Transparent |
| Border | 2px solid `color-primary` |
| Text | `color-primary`, 14px, 600 weight |
| Border radius | 12px |

**Usage:** Add to Cart (on PDP alongside Buy Now), Change Address

### 7.3 Status Badge

| Status | Background | Text Color | Dot Color |
|--------|-----------|------------|-----------|
| Active / In Stock | green-100 | green-700 | green-600 |
| Only N left! | amber-100 | amber-700 | amber-500 |
| Out of Stock / Rejected | red-100 | red-700 | red-500 |
| Pending Review | amber-100 | amber-800 | amber-500 |
| Hidden / Suspended | gray-100 | gray-600 | gray-400 |

**Format:** `[colored dot] [Label text]` — inline-flex, gap-1, text-xs, font-semibold

### 7.4 Form Field — Text Input

| State | Border | Background |
|-------|--------|-----------|
| Default | 1px solid gray-300 | white |
| Focus | 1px solid blue-500 (ring) | white |
| Valid (on blur) | 1px solid green-500 | green-50 |
| Error (on blur) | 2px solid red-500 | red-50 |
| Disabled | 1px solid gray-200 | gray-50, cursor not-allowed |

**Padding:** 8px vertical, 12px horizontal (py-2 px-3)  
**Border radius:** 8px (rounded-lg)  
**Font size:** 14px (text-sm)  
**Error text:** 12px, red-600, appears 4px below field

**Validation timing:** Fire on `blur` event. Never fire on `focus`. Optional: re-validate on `input` after first failed blur.

### 7.5 Product Card — Two variants

**Mobile list row (375px):**
```
┌────────────────────────────────┐
│ [56×56 thumb] [Name]      [⋮] │
│              [₹price · Stock] │
│              [● Status badge] │
└────────────────────────────────┘
```

**Desktop table row (SCR-003 seller dashboard, ≥1024px):**
```
│ [48×48] Product name / SKU  │ ₹Price │ Stock │ ● Status │ Date  │ Actions │
```

| Property | Mobile | Desktop |
|----------|--------|---------|
| Padding | 12px (p-3) | 16px 20px (py-4 px-5) |
| Thumbnail | 56×56px, rounded-lg | 48×48px, rounded-lg |
| Name | text-sm font-semibold, 1 line | text-sm font-medium, 1 line |
| Price/Stock | text-xs gray-500 | Separate table columns |
| Actions | Kebab menu (⋮) 44px | Edit / Deactivate text buttons, right-aligned |
| Divider | 1px border-b border-gray-100 | `<tbody> divide-y divide-gray-100` |

**Desktop Cart Item Row (SCR-002):**
- Thumbnail: 112×112px (w-28 h-28), rounded-xl, CloudFront CDN
- Layout: flex row with image left, product info + stepper right
- Remove button: ghost, 🗑 icon, red hover

### 7.6 Skeleton Loader

| Property | Value |
|----------|-------|
| Background | Shimmer gradient: `linear-gradient(90deg, #E5E7EB 25%, #F3F4F6 50%, #E5E7EB 75%)` |
| Animation | `background-size: 200%`, left-to-right, 1.5s infinite |
| Border radius | Match element it replaces (4px for text lines, 8px for images) |

**Rule:** Show skeleton for any content that takes > 200ms to load. Never show a spinner for page-level content loads.

### 7.7 Alert Banner

| Type | Background | Border | Icon |
|------|-----------|--------|------|
| Warning | amber-50 | border-b-2 amber-400 | ⚠️ |
| Error | red-50 | border-b-2 red-400 | ❌ |
| Info | blue-50 | border blue-200 | ℹ️ |
| Success | green-50 | border green-200 | ✅ |

**Behavior:** Full width, cannot be dismissed if it relates to account state (e.g., subscription suspension). Temporary alerts (cart updated) use toast pattern (auto-dismiss 4s).

### 7.8 Annotation Callout (Wireframe Only — Not Shipped)

| Property | Value |
|----------|-------|
| Shape | Circle, 20×20px |
| Background | #FBBF24 (amber-400) |
| Text | 11px, 700 weight, gray-900 |
| Usage | Wireframe annotations only — remove before production |

### 7.9 Seller Navigation — Two variants (responsive)

**Mobile (< 1024px): Bottom Navigation Bar**

| Property | Value |
|----------|-------|
| Height | 56px |
| Background | White |
| Border | 1px solid gray-200 top |
| Active tab | Blue-600 text + icon; 2px blue-600 border-top |
| Inactive tab | Gray-400 |
| Tap target | Full tab width, minimum 44px height |
| Icon size | 20px (text-xl emoji or SVG) |
| Label | 10px (text-xs) |

**Desktop (≥ 1024px): Left Sidebar Navigation — SCR-003 (A-05-009)**

| Property | Value |
|----------|-------|
| Width | 240px fixed (flex-shrink-0) |
| Background | White |
| Border | 1px solid gray-200, rounded-xl |
| Padding | 16px (p-4) |
| Nav item height | 40px (py-2.5 px-4), rounded-lg |
| Active item | bg-blue-50, text-blue-700, font-weight 600 |
| Inactive item | text-gray-500, hover: bg-gray-50 text-gray-900 |
| Icon | Emoji, 16px, aria-hidden="true" |
| Label | 14px (text-sm) |
| Keyboard | Tab to focus item, Enter to navigate, aria-current="page" on active |
| Tabs | Dashboard · Products · Orders · Payouts + divider + Settings · Subscription |

### 7.10 Quantity Stepper

| Property | Value |
|----------|-------|
| Layout | [−] [value] [+] in a flex row |
| Button size | Minimum 44×36px tap target |
| Border | 1px gray-300, rounded-lg, overflow hidden |
| Value display | Minimum width 40px, centered |
| Disabled (out of stock) | Opacity 40%, pointer-events none |
| Max | Capped at `stock_quantity` from API |

---

## 8. Iconography

**Strategy:** Emoji icons for MVP (zero load time, universal rendering on mobile OS).  
**Post-MVP:** Replace with Heroicons or Lucide React SVGs for consistency.

| Context | Icon | Notes |
|---------|------|-------|
| Cart | 🛒 | Top nav; count badge overlaid |
| Success | ✅ | Order success, valid stock |
| Error | ❌ | Out of stock, payment failure |
| Warning | ⚠️ | Suspension, cart update |
| Product | 📦 | Empty product state |
| Store | 🏪 | Seller store info on PDP |
| Shipping | 🚚 | Shipping info section |
| Hidden | 👁 | Suspended product label |
| Dashboard | 🏠 | Bottom nav / sidebar |
| Orders | 📋 | Bottom nav / sidebar |
| Profile | 👤 | Bottom nav / sidebar |
| Delete | 🗑️ | Remove from cart |
| Payouts | 💰 | Seller payouts nav |
| Settings | ⚙️ | Seller settings nav |
| Subscription | 💳 | Seller subscription nav |
| Search | 🔍 | Search button mobile |
| Lock | 🔒 | Security trust signals; masked bank details |
| Image upload | 📷 | File upload zone |
| Sellers (admin) | 👥 | Admin seller queue nav |

---

## 11. Additional Component Specifications (Full Run — SCR-004 through SCR-021)

### 11.1 Product Card — Buyer Browse (SCR-004, SCR-005, SCR-006)

```
Desktop grid card (4-col, ~280px wide):
┌──────────────────────────┐
│ [Image — 4:3 ratio]      │  ← CloudFront CDN; object-fit: cover
│                          │
│ Product Name             │  ← 2-line clamp; text-sm font-semibold
│ Seller Name              │  ← 1-line clamp; text-xs gray-500
│ ₹999.00                  │  ← text-sm font-bold; ₹ prefix, 2dp
└──────────────────────────┘
```

| Property | Value |
|----------|-------|
| Border radius | 12px (rounded-xl) |
| Border | 1px solid gray-200 |
| Hover shadow | Level 2 (0 4px 16px rgba(0,0,0,0.12)) |
| Image aspect ratio | 4:3 (padding-top: 75%) |
| Name lines | 2-line clamp (webkit-line-clamp: 2) |
| Transition | box-shadow 200ms ease |
| Full card clickable | `<a>` wraps entire card; `aria-label="{name}, ₹{price}"` |

### 11.2 Buyer Top Navigation Bar

| Property | Value |
|----------|-------|
| Height | 64px |
| Background | white |
| Border bottom | 1px solid gray-200 |
| Shadow | Level 1 (0 1px 3px rgba(0,0,0,0.08)) |
| Logo | font-size 20px, font-weight 800, color #1A56DB |
| Search bar | flex-grow 1; border gray-300; rounded-lg; padding 8px 16px; background gray-50 |
| Cart icon | 🛒 emoji; badge: 18×18px circle, bg orange-500, white text, font-size 10px |
| Max content width | 1280px (max-w-screen-xl mx-auto) |

### 11.3 Admin Navigation Shell

**Admin sidebar** uses a dark background to visually distinguish it from seller portal:

| Property | Value |
|----------|-------|
| Sidebar width | 240px fixed |
| Background | #1E293B (slate-800) — dark navy |
| Text (inactive) | #94A3B8 (slate-400) |
| Text (active) | white |
| Active item bg | #334155 (slate-700) |
| Hover bg | #334155 |
| Header background | #1E293B — matches sidebar |
| Logo text | white with orange accent on "Nest" |

**Rationale:** Admin dark theme prevents accidental confusion between admin and seller portals — especially important when testing locally. Dark admin = internal tool; light sidebar = seller tool.

### 11.4 Wizard Progress Indicator (SCR-011)

```
Step 1 ──── Step 2 ──── Step 3
  ●           ○           ○
(active)   (future)   (future)

Completed steps show ✓ green dot
Active step shows blue filled dot
Future steps show gray empty circle
```

| Property | Value |
|----------|-------|
| Dot size | 32×32px |
| Active dot | bg-blue-600, white text (step number) |
| Done dot | bg-green-600, white ✓ |
| Future dot | bg-gray-200, gray-500 text |
| Connecting line | 2px solid gray-300 (done segment: green-500) |
| Step label | text-xs gray-600 below dot |
| ARIA | `role="progressbar"` with `aria-valuenow`, `aria-valuemax`, `aria-valuetext` |

### 11.5 Order Status Timeline (SCR-008)

```
[✅ Payment Confirmed] ──── [⏳ Processing] ──── [○ Shipped] ──── [○ Delivered]
```

| State | Indicator | Color |
|-------|-----------|-------|
| Completed | ✅ filled circle | green-600 |
| Current | ⏳ filled circle | amber-600 |
| Future | ○ empty circle | gray-300 |
| Cancelled | ✗ red circle | red-600 |

Connecting lines: green for completed segments, gray for future segments.
ARIA: `role="list"` on timeline; `aria-current="step"` on current step.

### 11.6 KPI Stat Card (SCR-013, SCR-021)

```
┌──────────────────────┐
│ ORDERS THIS MONTH    │  ← label: text-xs gray-500 uppercase tracking-wide
│                      │
│ 12                   │  ← value: text-4xl font-black gray-900
│ +18% vs last month   │  ← sub: text-xs gray-500
└──────────────────────┘
```

| Property | Value |
|----------|-------|
| Border | 1px solid gray-200 |
| Border radius | 12px (rounded-xl) |
| Padding | 20px |
| Background | white |
| Label | text-xs, font-semibold, uppercase, tracking-wide, gray-500 |
| Value | text-4xl, font-black, gray-900, line-height 1.1 |
| Sub | text-xs, gray-500, margin-top 4px |
| ARIA | `aria-label="{label}: {value} — {sub}"` on article element |

### 11.7 Data Table (Seller Orders, Payouts, Admin Queues)

| Property | Value |
|----------|-------|
| Header row bg | gray-50 |
| Header text | text-xs, font-semibold, uppercase, tracking-wide, gray-500 |
| Header border | border-bottom 1px gray-200 |
| Cell padding | 14px 16px |
| Row border | border-bottom 1px gray-100 |
| Row hover | bg-gray-50 |
| Expandable row | bg-blue-50 / gray-50 for expanded state |
| Rounded container | rounded-xl border border-gray-200 overflow-hidden |
| ARIA | `role="table"` · `scope="col"` on `<th>` · `aria-expanded` on expandable rows |

### 11.8 Category / Wizard Multi-Select Chips

```
[✓ Women's Clothing]  [  Men's Clothing  ]  [  Electronics  ]
   (selected — blue)       (unselected)          (unselected)
```

| State | Background | Text | Border |
|-------|-----------|------|--------|
| Unselected | white | gray-700 | 1px gray-300 |
| Selected | blue-50 | blue-700 | 2px blue-600 |
| Hover | gray-50 | gray-900 | 1px gray-400 |

Padding: 8px 16px · Border radius: 9999px (rounded-full) · Font size: 14px · Min height: 44px (WCAG 2.5.5)
ARIA: `role="checkbox"` `aria-checked="true/false"` inside `role="group"` fieldset.

### 11.9 File Upload Zone (SCR-011, SCR-003)

```
┌──────────────────────────────────┐
│  📷                              │  ← emoji, text-2xl, gray-400
│  Drag & drop or click to upload  │  ← text-sm gray-600
│  JPG · PNG · Max 2MB             │  ← text-xs gray-400
└──────────────────────────────────┘
```

| State | Border | Background |
|-------|--------|-----------|
| Default | 2px dashed gray-300 | gray-50 |
| Drag over | 2px dashed blue-500 | blue-50 |
| Error | 2px dashed red-400 | red-50 |
| Uploaded | 1px solid green-400 | green-50 |

Implementation: `<input type="file">` is the accessible baseline. Drag-drop is a JS enhancement layered on top. The label element associated with the input is the upload zone. `accept="image/jpeg,image/png,image/webp"`.

---

## 9. Responsive Breakpoints

| Breakpoint | Width | Layout Change |
|-----------|-------|--------------|
| Mobile (default) | 375px+ | Single column; buyer top nav; seller: bottom nav; full-width buttons |
| Tablet | 768px+ | 2-column product grid; cart: side summary panel appears |
| Desktop | 1024px+ | Seller: 240px left sidebar replaces bottom nav; buyer: full 2-col cart layout; PDP: 2-col image+info |
| Wireframe reference | 1280px+ | HTML files viewport-locked at `min-width: 1280px` for review; not a production breakpoint |
| Wide desktop | 1440px | Wireframe reference frame; `max-w-screen-xl mx-auto` constrains content to 1280px |

**Rule:** Mobile layout is the primary implementation target. Desktop enhancements are additive — never remove mobile-designed functionality at larger breakpoints. Wireframe HTML files use 1440px reference frame for inspection clarity only.

---

## 10. Motion & Animation

| Context | Spec |
|---------|------|
| Skeleton shimmer | 1.5s linear infinite; subtle (not distracting) |
| Cart count badge update | Fade-in 200ms; no bounce |
| Alert banner appearance | Slide-down 200ms ease-out |
| Button press feedback | Scale 0.97 on active (CSS transform) |
| Page transitions | None at MVP (Next.js default) |
| Error field shake | None — errors appear inline without animation |

**Reduced motion:** All animations respect `prefers-reduced-motion: reduce` media query — set `animation: none` and `transition: none` when enabled.
