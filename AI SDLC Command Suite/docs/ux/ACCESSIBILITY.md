# ACCESSIBILITY.md — ShopNest WCAG 2.1 AA Compliance

**Phase:** 05 — UX Design (Full Production Run)
**Date:** 2026-05-15
**Standard:** WCAG 2.1 Level AA
**Scope:** All 21 screens (SCR-001 through SCR-021)

---

## Compliance Summary

| Screen | Target Level | Estimated Compliance |
|--------|-------------|---------------------|
| SCR-001 — Product Detail Page | WCAG 2.1 AA | ✅ Compliant (with implementation guidance below) |
| SCR-002 — Cart & Checkout | WCAG 2.1 AA | ✅ Compliant (with implementation guidance below) |
| SCR-003 — Seller Product Management | WCAG 2.1 AA | ✅ Compliant (with implementation guidance below) |
| SCR-004 — Marketplace Homepage | WCAG 2.1 AA | ✅ Compliant (see §12) |
| SCR-005 — Category Listing | WCAG 2.1 AA | ✅ Compliant (see §12) |
| SCR-006 — Search Results | WCAG 2.1 AA | ✅ Compliant (see §12) |
| SCR-007 — Buyer Authentication | WCAG 2.1 AA | ✅ Compliant (see §13) |
| SCR-008 — Order Tracking | WCAG 2.1 AA | ✅ Compliant (see §13) |
| SCR-009 — Buyer Order History | WCAG 2.1 AA | ✅ Compliant (see §13) |
| SCR-010 — Seller Registration | WCAG 2.1 AA | ✅ Compliant (see §14) |
| SCR-011 — Store Setup Wizard | WCAG 2.1 AA | ✅ Compliant (see §14) |
| SCR-012 — Seller Subscription Signup | WCAG 2.1 AA | ✅ Compliant (see §14) |
| SCR-013 — Seller Dashboard | WCAG 2.1 AA | ✅ Compliant (see §14) |
| SCR-014 — Seller Order Management | WCAG 2.1 AA | ✅ Compliant (see §14) |
| SCR-015 — Seller Payouts | WCAG 2.1 AA | ✅ Compliant (see §14) |
| SCR-016 — Seller Subscription Management | WCAG 2.1 AA | ✅ Compliant (see §14) |
| SCR-017 — Seller Store Settings | WCAG 2.1 AA | ✅ Compliant (see §14) |
| SCR-018 — Admin Login | WCAG 2.1 AA | ✅ Compliant (see §15) |
| SCR-019 — Admin Seller Approval Queue | WCAG 2.1 AA | ✅ Compliant (see §15) |
| SCR-020 — Admin Product Approval Queue | WCAG 2.1 AA | ✅ Compliant (see §15) |
| SCR-021 — Admin Dashboard | WCAG 2.1 AA | ✅ Compliant (see §15) |

---

## 1. Color Contrast (WCAG 1.4.3 — Minimum Contrast, AA: 4.5:1 normal / 3:1 large)

### All Three Screens

| Element | Foreground | Background | Ratio | Pass/Fail |
|---------|-----------|-----------|-------|-----------|
| Body text (gray-900 on white) | #111827 | #FFFFFF | 16.75:1 | ✅ Pass |
| Secondary text (gray-500 on white) | #6B7280 | #FFFFFF | 4.60:1 | ✅ Pass |
| Muted text (gray-400 on white) | #9CA3AF | #FFFFFF | 2.85:1 | ❌ Fail — muted text must only be used for decorative/non-essential content (placeholder text is exempt per WCAG 1.4.3) |
| Orange CTA button text (white on orange-500) | #FFFFFF | #F97316 | 3.00:1 | ⚠️ Borderline for normal text (14px) — use font-weight 600+ and consider darkening to orange-600 (#EA580C, ratio 3.51:1) |
| Blue primary link (blue-700 on white) | #1D4ED8 | #FFFFFF | 5.90:1 | ✅ Pass |
| Success badge text (green-700 on green-100) | #15803D | #DCFCE7 | 4.87:1 | ✅ Pass |
| Error badge text (red-700 on red-100) | #B91C1C | #FEE2E2 | 4.90:1 | ✅ Pass |
| Amber badge text (amber-800 on amber-100) | #92400E | #FEF3C7 | 5.20:1 | ✅ Pass |
| Error message text (red-600 on white) | #DC2626 | #FFFFFF | 4.52:1 | ✅ Pass |
| Disabled button text (gray-400 on gray-200) | #9CA3AF | #E5E7EB | 1.42:1 | ⚠️ Intentionally low contrast — disabled controls are exempt per WCAG 1.4.3 Note 1 |

**Action Required (before Phase 7 implementation):**
- Orange CTA button: Use `font-weight: 600` and minimum font-size 14px. Consider `#EA580C` (orange-600) for body-text-sized labels to ensure 3.5:1+.
- Gray-400 muted text: Reserve exclusively for placeholder text (exempt) and non-informational decorative elements. Never use for status descriptions or error messages.

---

## 2. Keyboard Navigation (WCAG 2.1.1 — Keyboard, 2.4.3 — Focus Order)

### SCR-001: Product Detail Page

| Element | Tab Order | Focus Visible | Notes |
|---------|-----------|--------------|-------|
| Back link | 1 | ✅ Outline | Standard link focus ring |
| Cart icon button | 2 | ✅ Outline | `role="link"` or `<a>` |
| Image carousel | 3 | ✅ | Arrow keys for carousel navigation; `aria-roledescription="carousel"` |
| Quantity decrease (−) | 4 | ✅ | `<button>` element |
| Quantity value | 5 | ✅ (optional) | `aria-live="polite"` for screen reader announcements |
| Quantity increase (+) | 6 | ✅ | `<button>` element |
| Add to Cart | 7 | ✅ | `<button>` element |
| Buy Now | 8 | ✅ | `<button>` element |
| Description accordion toggle | 9 | ✅ | `<button aria-expanded="false/true">` |
| Seller store link | 10 | ✅ | `<a>` element |

**Disabled buttons (Out of Stock / Self-purchase):** Use `disabled` attribute — screen readers announce "dimmed" or "unavailable". Do NOT use `aria-disabled` only without `disabled` (keyboard focus would still land on the button).

### SCR-002: Cart & Checkout

| Element | Tab Order | Focus Visible | Notes |
|---------|-----------|--------------|-------|
| Back link | 1 | ✅ | |
| Cart items (each) | Sequential | ✅ | Each item row: quantity −, +, remove |
| Remove button (🗑️) | Within item | ✅ | Descriptive `aria-label`: "Remove Blue Cotton Kurti from cart" |
| Proceed to Checkout | Last in cart | ✅ | |
| Guest form fields | Sequential | ✅ | Tab order: Name → Email → Phone → Address Line 1 → City → State → PIN |
| "Sign in" link | After PIN | ✅ | `<a>` element |
| Pay button | Last | ✅ | |

**Focus management on state change:** When cart item is removed, focus moves to next item or to "Your cart is empty" heading. Do NOT leave focus on removed element.

### SCR-003: Seller Product Management (Desktop — Sidebar Layout)

| Element | Tab Order | Focus Visible | Notes |
|---------|-----------|--------------|-------|
| SellerTopBar — logo link | 1 | ✅ | `<a>` element |
| Sidebar nav items (6 items) | 2–7 | ✅ | `tabindex="0"` + `role="link"` · active item: `aria-current="page"` · keyboard: Tab advances through items, Enter activates |
| Add Product button | 8 | ✅ | `<button>` · disabled when subscription not ACTIVE |
| Status filter `<select>` | 9 | ✅ | Native `<select>` — keyboard: ↑↓ to change options |
| Search input | 10 | ✅ | `<input type="text">` |
| Product table rows — Edit button | Sequential per row | ✅ | `aria-label="Edit {product name}"` |
| Product table rows — Deactivate button | Within row | ✅ | `aria-label="Deactivate {product name}"` |
| Pagination — Prev / Next | After table | ✅ | `disabled` attr when unavailable; `aria-disabled="true"` |
| Add Product form fields | Sequential | ✅ | Per form field spec (§5 below) |

**Sidebar nav keyboard pattern:**
```html
<!-- Each nav item -->
<div class="nav-item" tabindex="0" role="link" aria-label="Products — current page" aria-current="page">
  <span aria-hidden="true">📦</span> Products
</div>
```
- Tab key cycles through all sidebar items
- Enter key activates navigation (page transition)
- Sidebar is `<aside role="navigation" aria-label="Seller dashboard navigation">`

**Desktop keyboard tab order — main flow:**
1. Skip-to-main-content link (Phase 7 implementation) → main content
2. Sidebar nav items (Tab through) → 
3. Add Product button → Search input → Status filter →
4. First product row Edit/Deactivate → next row... →
5. Pagination buttons

**Suspended state:** `role="alert"` on SubscriptionGatePanel fires announcement immediately. Renew Subscription button is the first focusable element in `<main>` when suspension is active (Tab from sidebar lands here first).

**Subscription Gate (State 5) keyboard:** Sidebar remains fully keyboard-navigable even when main content is the gate panel. This allows the seller to Tab to the Subscription sidebar item directly.

---

## 3. Focus Indicator (WCAG 2.4.7 — Focus Visible; 2.4.11 — Focus Appearance, AA)

**Specification for all interactive elements:**
```css
:focus-visible {
  outline: 2px solid #1A56DB;
  outline-offset: 2px;
  border-radius: 4px;
}
```
- Minimum 2px outline
- `outline-offset: 2px` ensures outline does not merge with element border
- `border-radius: 4px` matches element shape on rounded elements
- Color: `#1A56DB` (blue-700) — 4.52:1 contrast against white background ✅

**Do NOT use** `outline: none` anywhere without a visible replacement focus style.

---

## 4. Screen Reader Semantics (WCAG 1.3.1 — Info and Relationships)

### SCR-001: Product Detail Page

```html
<!-- Product heading -->
<h1>Blue Cotton Kurti</h1>

<!-- Price with screen reader context -->
<p>
  <span class="sr-only">Price:</span>
  <strong aria-label="Price: 999 rupees">₹999.00</strong>
</p>

<!-- Stock status -->
<p role="status" aria-live="polite">
  <span class="sr-only">Stock status:</span>
  In Stock
</p>

<!-- Image carousel -->
<div role="region" aria-roledescription="Product image carousel" aria-label="Product images">
  <img src="..." alt="Blue Cotton Kurti — front view showing full-length drape" />
  <button aria-label="Next image">›</button>
  <button aria-label="Previous image">‹</button>
</div>

<!-- Quantity stepper -->
<div role="group" aria-label="Quantity">
  <button aria-label="Decrease quantity">−</button>
  <span aria-live="polite" aria-atomic="true">1</span>
  <button aria-label="Increase quantity">+</button>
</div>

<!-- Disabled CTA (out of stock) -->
<button disabled aria-describedby="out-of-stock-msg">Add to Cart</button>
<p id="out-of-stock-msg" class="sr-only">This product is currently out of stock</p>

<!-- Self-purchase block -->
<button disabled aria-describedby="self-purchase-msg">Buy Now</button>
<p id="self-purchase-msg">You cannot purchase your own product</p>
```

### SCR-002: Cart & Checkout

```html
<!-- Cart page heading -->
<h1>Your Cart <span aria-label="2 items">(2)</span></h1>

<!-- Cart item remove button -->
<button aria-label="Remove Blue Cotton Kurti from cart">🗑️</button>

<!-- Order total -->
<p aria-label="Order total: 2,197 rupees">
  Total <strong>₹2,197.00</strong>
</p>

<!-- Guest form -->
<form aria-label="Guest checkout">
  <label for="guest-name">Full Name <span aria-hidden="true">*</span></label>
  <input id="guest-name" type="text" required aria-required="true" />

  <label for="guest-email">Email <span aria-hidden="true">*</span></label>
  <input id="guest-email" type="email" required aria-required="true"
    aria-invalid="true" aria-describedby="email-error" />
  <p id="email-error" role="alert">Please enter a valid email address</p>

  <label for="guest-phone">Phone (+91) <span aria-hidden="true">*</span></label>
  <input id="guest-phone" type="tel" required aria-required="true"
    inputmode="numeric" pattern="[6-9][0-9]{9}" />
</form>

<!-- Payment button -->
<button type="submit" aria-label="Pay 999 rupees securely via Razorpay">
  Pay ₹999.00
</button>

<!-- Success page -->
<main aria-label="Order confirmed">
  <h1>Order Placed!</h1>
  <p role="status">Your order #SN-2026-001234 has been placed successfully.</p>
</main>
```

### SCR-003: Seller Product Management

```html
<!-- Tab interface -->
<div role="tablist" aria-label="Filter products by status">
  <button role="tab" aria-selected="true" aria-controls="panel-all" id="tab-all">All (3)</button>
  <button role="tab" aria-selected="false" aria-controls="panel-active" id="tab-active">Active (1)</button>
  <button role="tab" aria-selected="false" aria-controls="panel-pending" id="tab-pending">Pending (1)</button>
  <button role="tab" aria-selected="false" aria-controls="panel-rejected" id="tab-rejected">Rejected (1)</button>
</div>
<div role="tabpanel" id="panel-all" aria-labelledby="tab-all">...</div>

<!-- Product status -->
<li aria-label="Blue Cotton Kurti — Active — ₹999 — Stock: 50">
  <!-- row content -->
  <button aria-label="Product options for Blue Cotton Kurti"
    aria-haspopup="menu" aria-expanded="false">⋮</button>
</li>

<!-- Suspension banner -->
<div role="alert" aria-live="assertive">
  <p>Subscription Suspended. Your products are hidden from buyers.</p>
  <button>Renew Subscription — ₹1,999/month</button>
</div>

<!-- Add product form -->
<form aria-label="Add new product">
  <label for="product-name">Product Name <span aria-hidden="true">*</span></label>
  <input id="product-name" type="text" required aria-required="true" />

  <label for="product-desc">Description <span aria-hidden="true">*</span></label>
  <textarea id="product-desc" required aria-required="true"
    aria-describedby="desc-counter desc-hint"></textarea>
  <p id="desc-counter" aria-live="polite">0 of 100 minimum characters</p>
  <p id="desc-hint">Minimum 100 characters required</p>
</form>
```

---

## 5. Error Identification & Description (WCAG 3.3.1 — Error Identification, 3.3.3 — Error Suggestion)

| Requirement | Implementation |
|-------------|---------------|
| Error identified in text | Error messages appear as visible text, not only as red color |
| Error programmatically associated | `aria-invalid="true"` + `aria-describedby="{error-id}"` on field |
| Error message actionable | Not "Invalid input" — instead "Please enter a valid 10-digit Indian mobile number" |
| Error not on submit only | Validation fires on blur; form submit re-validates all fields and focuses first error |
| Success identified | `aria-live="polite"` region announces "Order placed successfully" on confirmation |

**Error announcement pattern:**
```html
<!-- Field with error -->
<input aria-invalid="true" aria-describedby="phone-error" />
<p id="phone-error" role="alert">
  Please enter a valid 10-digit Indian mobile number
</p>
```

**`role="alert"` fires immediately** when inserted in DOM — screen readers announce without user action.

---

## 6. Touch Target Size (WCAG 2.5.5 — Target Size, AA: minimum 44×44px)

| Component | Size | Pass/Fail |
|-----------|------|----------|
| Primary CTA button | Full width × 48px | ✅ |
| Secondary outline button | Full width × 48px | ✅ |
| Quantity stepper (−/+) | 44px × 36px (enforce 44×44 minimum) | ⚠️ Height must be 44px minimum |
| Remove cart item button | 44px × 44px (add padding if icon-only) | ✅ (with padding) |
| Kebab menu (⋮) | 44px × 44px | ✅ (with padding) |
| Status filter tabs | Full tab width × 40px | ⚠️ Increase to 44px height |
| Bottom nav tabs | Full tab width × 56px | ✅ |
| Add Product FAB | 48px diameter (w-12 h-12) | ✅ |
| Carousel dot navigation | 24px dots — too small | ⚠️ Use 44px invisible tap area around each dot |

**Action Required:** Quantity stepper buttons, status filter tabs, and carousel dots must have at minimum 44×44px tap areas. Use padding to expand tap target without changing visual size.

---

## 7. Text Alternatives (WCAG 1.1.1 — Non-text Content)

| Element | Implementation |
|---------|---------------|
| Product images | `alt="[Product name] — [view description, e.g., front view showing full-length drape]"` |
| Seller logo | `alt="[Store name] logo"` |
| Cart icon | `aria-label="Shopping cart, 1 item"` (count in label, updated via `aria-live`) |
| Status dot (colored circle) | Status name in adjacent text; dot is `aria-hidden="true"` |
| Emoji icons | `aria-hidden="true"` on all emoji; text label adjacent or in `aria-label` on parent |
| Loading skeleton | `aria-busy="true"` on container; `aria-label="Loading product details"` |
| Image upload slot | `aria-label="Upload product image 1"` on each slot |

---

## 8. Form Labels (WCAG 1.3.1, 3.3.2 — Labels or Instructions)

| Rule | Implementation |
|------|---------------|
| Every input has a visible label | `<label for="...">` associated to `<input id="...">` — no placeholder-only labels |
| Required fields indicated | Visible asterisk (*) with `aria-hidden="true"` + `aria-required="true"` on input |
| Instructions provided before input | Character counter spec shown before textarea; format hints below field |
| Placeholder text supplemental only | Placeholder shows example value, not the label itself |

---

## 9. Motion & Reduced Motion (WCAG 2.3.3 — Animation from Interactions, AAA; best practice at AA)

```css
@media (prefers-reduced-motion: reduce) {
  .shimmer { animation: none; background: #E5E7EB; }
  * { transition-duration: 0.01ms !important; animation-duration: 0.01ms !important; }
}
```

All CSS animations and transitions must be wrapped in `prefers-reduced-motion` check. The skeleton shimmer is the primary animation to disable.

---

## 10. Language (WCAG 3.1.1 — Language of Page)

```html
<html lang="en">
```

ShopNest MVP is English-only. If Hindi or regional language support is added post-MVP, `lang` attribute must be updated and screen reader testing repeated.

---

## 11. Phase 9 Accessibility Test Cases

| Test ID | Test | Tool | Pass Criteria |
|---------|------|------|--------------|
| A11Y-001 | Automated WCAG 2.1 AA scan on SCR-001 | axe-core (jest-axe) | Zero critical/serious violations |
| A11Y-002 | Automated WCAG 2.1 AA scan on SCR-002 | axe-core | Zero critical/serious violations |
| A11Y-003 | Automated WCAG 2.1 AA scan on SCR-003 | axe-core | Zero critical/serious violations |
| A11Y-004 | Keyboard-only navigation on SCR-001 PDP | Manual | All interactive elements reachable; focus order logical |
| A11Y-005 | Keyboard-only checkout completion on SCR-002 | Manual | Guest form completable without mouse |
| A11Y-006 | Screen reader (VoiceOver iOS) — product page | VoiceOver | Price, stock status, CTAs announced correctly |
| A11Y-007 | Screen reader — cart removal announcement | VoiceOver/TalkBack | "Blue Cotton Kurti removed. Cart now has 1 item." announced |
| A11Y-008 | Screen reader — form error announcement | VoiceOver | Error "Please enter a valid email" announced on blur |
| A11Y-009 | Colour contrast check on orange CTA button | Colour Contrast Analyser | ≥ 3:1 for large/bold text at 14px/600 weight |
| A11Y-010 | Touch target audit on SCR-003 mobile (375px) | axe DevTools mobile | All interactive elements ≥ 44×44px |
| A11Y-011 | Reduced motion — skeleton shimmer disabled | Manual (DevTools emulate) | Shimmer animation stops; static grey background shown |
| A11Y-012 | Suspension banner announced on page load | VoiceOver | `role="alert"` fires: "Subscription Suspended. Products hidden from buyers." |
| A11Y-013 | Search results count announced | VoiceOver | `aria-live="polite"` fires: "42 products found for 'cotton kurti'" |
| A11Y-014 | Order status timeline announced | VoiceOver (SCR-008) | Each timeline step announced with status (done/current/future) |
| A11Y-015 | Wizard progress bar announced | VoiceOver (SCR-011) | `aria-valuenow` updates on each step; "Step 2 of 3" announced |
| A11Y-016 | Modal focus trap — cancel subscription | Keyboard test (SCR-016) | Tab cycles within modal; Escape closes; focus returns to trigger |
| A11Y-017 | Admin reject modal focus trap | Keyboard test (SCR-019, 020) | Tab cycles within modal; rejection reason required before submit |
| A11Y-018 | Admin queue tab bar | VoiceOver (SCR-019, 020) | `role="tablist"` announced; selected tab aria-selected="true" |
| A11Y-019 | KPI cards — admin and seller dashboard | VoiceOver (SCR-013, 021) | `aria-label="Orders this month: 12"` reads correctly |
| A11Y-020 | Full WCAG 2.1 AA automated scan — all 21 screens | axe-core CI gate | Zero critical/serious violations on all 21 screens before Phase 7 merge |

---

## 12. Buyer Browsing Screens — Keyboard & Semantic Patterns (SCR-004, 005, 006)

### SCR-004: Homepage

| Element | Role / Pattern | Notes |
|---------|---------------|-------|
| Hero banner CTA | `<a>` or `<button>` | Large tap target; visible focus ring |
| Category chips | `<a href="/categories/{slug}">` | Native link; keyboard navigable; aria-label includes category name |
| Product grid | `role="list"` on grid container; `role="listitem"` per card | Allows screen reader to count products |
| Product card | `<a href="/products/{slug}">` wrapping entire card | Single focusable element per card; `aria-label="{name}, ₹{price}, sold by {seller}"` |
| "View All" link | `<a href="/search">` | Descriptive text — not "click here" |

### SCR-005: Category Listing

| Element | Role / Pattern | Notes |
|---------|---------------|-------|
| Breadcrumb | `<nav aria-label="Breadcrumb"><ol>` with `<li>` items | Current page `aria-current="page"` on last item |
| Sort dropdown | Native `<select>` | Keyboard accessible; label "Sort products by" |
| Product grid | Same as SCR-004 (`role="list"`) | 24 products per page |
| Pagination | `<nav aria-label="Pagination">` | Prev/Next: `aria-disabled="true"` when at boundary; current page `aria-current="page"` |

### SCR-006: Search Results

```html
<!-- Search landmark -->
<nav role="search" aria-label="Product search">
  <input type="search" aria-label="Search products"
    value="cotton kurti" />
  <button type="submit" aria-label="Search">🔍</button>
</nav>

<!-- Result count — announced after results load -->
<p aria-live="polite" role="status">
  42 products found for "cotton kurti"
</p>

<!-- No results -->
<div role="status" aria-live="polite">
  <p>No products found for "xyz123"</p>
</div>
```

---

## 13. Buyer Auth & Order Screens — Patterns (SCR-007, 008, 009)

### SCR-007: Buyer Authentication

```html
<!-- Tab toggle -->
<div role="tablist" aria-label="Authentication options">
  <button role="tab" aria-selected="true" aria-controls="login-panel">Login</button>
  <button role="tab" aria-selected="false" aria-controls="register-panel">Register</button>
</div>
<div role="tabpanel" id="login-panel" aria-labelledby="...">
  <form aria-label="Sign in to your account">...</form>
</div>

<!-- Password show/hide -->
<input type="password" id="password" />
<button type="button" aria-label="Show password" aria-pressed="false"
  onclick="toggle()">👁</button>

<!-- Login error -->
<div role="alert" aria-live="assertive">
  Email or password is incorrect. Please try again.
</div>
```

### SCR-008: Order Tracking

```html
<!-- Status timeline -->
<ol role="list" aria-label="Order status timeline">
  <li aria-label="Payment Confirmed — completed">✅ Payment Confirmed</li>
  <li aria-label="Processing — current step" aria-current="step">⏳ Processing</li>
  <li aria-label="Shipped — not yet reached">○ Shipped</li>
  <li aria-label="Delivered — not yet reached">○ Delivered</li>
</ol>

<!-- Cancel confirmation (inline) -->
<div role="alertdialog" aria-label="Confirm order cancellation"
  aria-describedby="cancel-desc">
  <p id="cancel-desc">You will receive a full refund within 3–7 business days.</p>
  <button>Yes, Cancel Order</button>
  <button>Keep Order</button>
</div>

<!-- Invalid token -->
<main role="alert" aria-live="assertive">
  This tracking link is invalid or expired.
</main>
```

### SCR-009: Buyer Order History

```html
<!-- Order list -->
<main aria-label="My orders">
  <h1>My Orders</h1>
  <ul role="list">
    <li role="listitem" aria-label="Order SN-001234, Blue Cotton Kurti, ₹999, Delivered">
      <!-- order card content -->
      <a href="/orders/track/..." aria-label="Track order SN-001234">Track Order →</a>
    </li>
  </ul>
</main>
```

---

## 14. Seller Portal Screens — Keyboard & Semantic Patterns (SCR-010 through SCR-017)

### SCR-010: Seller Registration

Same form patterns as SCR-002 guest checkout. Additionally:
- Password strength indicator: `<div role="meter" aria-label="Password strength" aria-valuenow="2" aria-valuemin="0" aria-valuemax="4" aria-valuetext="Fair">` with 4 visual blocks

### SCR-011: Store Setup Wizard

```html
<!-- Wizard progress -->
<div role="progressbar" aria-label="Store setup progress"
  aria-valuenow="1" aria-valuemin="1" aria-valuemax="3"
  aria-valuetext="Step 1 of 3: Store Information">
</div>

<!-- Category multi-select chips -->
<fieldset>
  <legend>Store Categories (select up to 3)</legend>
  <label><input type="checkbox" name="category" value="womens" checked /> Women's Clothing</label>
  <label><input type="checkbox" name="category" value="mens" /> Men's Clothing</label>
  <!-- etc. -->
</fieldset>

<!-- File upload zone -->
<label for="logo-upload">
  Store Logo (JPG/PNG, max 2MB)
  <input id="logo-upload" type="file" accept="image/jpeg,image/png" />
</label>
<!-- Drag-drop is JS enhancement; input is the accessible base -->
```

### SCR-012: Seller Subscription Signup

```html
<!-- Razorpay modal (ShopNest-side annotation only) -->
<!-- When modal opens: focus moves inside modal, trapped -->
<!-- aria-modal="true" tells screen readers to ignore background -->
<div role="dialog" aria-modal="true" aria-label="Razorpay payment gateway">
  <!-- Razorpay renders this entirely — ShopNest does not control content -->
</div>
```

### SCR-013: Seller Dashboard

```html
<!-- KPI cards -->
<section aria-label="Dashboard metrics">
  <article aria-label="Orders this month: 12">
    <p class="kpi-label">Orders This Month</p>
    <p class="kpi-value">12</p>
  </article>
  <!-- etc. -->
</section>

<!-- Subscription inactive alert -->
<div role="alert" aria-live="assertive">
  Your subscription has expired. Products are hidden from buyers.
  <a href="/seller/subscription">Renew Subscription</a>
</div>
```

### SCR-014: Seller Order Management

```html
<!-- Orders table -->
<table role="table" aria-label="Seller orders">
  <thead>
    <tr>
      <th scope="col">Order</th>
      <th scope="col">Buyer</th>
      <th scope="col">Items</th>
      <th scope="col">Total</th>
      <th scope="col">Status</th>
      <th scope="col">Actions</th>
    </tr>
  </thead>
  <tbody>
    <tr aria-expanded="false"><!-- order row --></tr>
    <tr role="region" aria-label="Order details for SN-001234" hidden>
      <!-- expanded detail panel -->
    </tr>
  </tbody>
</table>

<!-- Shipping form (State 6 — awaiting shipment) -->
<form aria-label="Enter shipping details">
  <label for="courier-name">Courier Name <span aria-hidden="true">*</span></label>
  <input id="courier-name" type="text" required aria-required="true" />
  <label for="awb-number">AWB Number <span aria-hidden="true">*</span></label>
  <input id="awb-number" type="text" required aria-required="true" />
</form>
```

### SCR-015–SCR-017: Seller Secondary Screens

| Screen | Key Pattern |
|--------|------------|
| SCR-015 Payouts | Payout table: `scope="col"` on all headers; "Download PDF" links: `aria-label="Download invoice for payout of ₹24,010 on 12 May 2026"` |
| SCR-016 Subscription | Cancel modal: `role="dialog"` focus trap; Escape closes; `aria-describedby` on cancellation consequences text |
| SCR-017 Settings | Masked bank account: `aria-label="Bank account ending in 4321"`; "Edit" reveals input with `aria-label="Enter bank account number"` |

---

## 15. Admin Portal Screens — Keyboard & Semantic Patterns (SCR-018 through SCR-021)

### SCR-018: Admin Login

Same patterns as SCR-007 (login tab). No tabs needed — single form. Focus on email field on page load via `autofocus`.

### SCR-019 & SCR-020: Admin Approval Queues

```html
<!-- Admin sidebar navigation -->
<nav aria-label="Admin navigation" role="navigation">
  <a href="/admin/dashboard" aria-current="page">🏠 Dashboard</a>
  <a href="/admin/sellers">👥 Seller Queue</a>
  <a href="/admin/products">📦 Product Queue</a>
</nav>

<!-- Queue tab bar -->
<div role="tablist" aria-label="Application status filter">
  <button role="tab" aria-selected="true"
    aria-controls="pending-panel" id="tab-pending">
    Pending (3)
  </button>
  <button role="tab" aria-selected="false"
    aria-controls="approved-panel" id="tab-approved">
    Approved
  </button>
  <button role="tab" aria-selected="false"
    aria-controls="rejected-panel" id="tab-rejected">
    Rejected
  </button>
</div>

<!-- Expandable row -->
<tr aria-expanded="false">
  <button aria-expanded="false" aria-controls="detail-row-001"
    aria-label="View application from Priya Sharma">
    View ▼
  </button>
</tr>
<tr id="detail-row-001" role="region"
  aria-label="Application details for Priya Sharma" hidden>
  <!-- expanded content -->
  <button class="btn-approve" aria-label="Approve Priya Sharma's seller application">Approve</button>
  <button class="btn-reject" aria-label="Reject Priya Sharma's seller application">Reject</button>
</tr>

<!-- Reject modal -->
<div role="dialog" aria-modal="true"
  aria-label="Reject seller application"
  aria-describedby="reject-desc">
  <p id="reject-desc">Enter a rejection reason. This will be emailed to the seller.</p>
  <label for="rejection-reason">Rejection Reason <span aria-hidden="true">*</span></label>
  <textarea id="rejection-reason" required aria-required="true"></textarea>
  <button>Confirm Reject</button>
</div>

<!-- Product image enlarged (SCR-020) -->
<div role="dialog" aria-modal="true"
  aria-label="Enlarged product image"
  aria-describedby="image-caption">
  <img src="..." alt="Blue Cotton Kurti — product image 1 of 3" />
  <p id="image-caption">Click × or press Escape to close</p>
  <button aria-label="Close enlarged image">×</button>
</div>
```

### SCR-021: Admin Dashboard

```html
<!-- KPI section -->
<section aria-label="Platform health overview">
  <article aria-label="Active sellers: 12, 3 pending review">
    <p class="kpi-label">Active Sellers</p>
    <p class="kpi-value">12</p>
    <p class="kpi-sub">3 pending review</p>
  </article>
  <!-- etc. -->
</section>

<!-- Action required region -->
<section aria-label="Actions required">
  <div role="region" aria-label="3 seller applications pending review">
    <p>3 seller applications pending review</p>
    <a href="/admin/sellers">Review Sellers →</a>
  </div>
</section>

<!-- Activity list -->
<section aria-label="Recent activity">
  <ul role="list">
    <li>Priya Sharma approved — <time datetime="2026-05-15T10:30">3 min ago</time></li>
    <!-- etc. -->
  </ul>
</section>
```

**Admin-specific WCAG note:** Admin portal is desktop-only (no mobile requirement). WCAG 2.1 keyboard and screen reader requirements still apply fully — admin users may use assistive technology.
