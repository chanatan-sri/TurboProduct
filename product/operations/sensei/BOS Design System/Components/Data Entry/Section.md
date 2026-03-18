# Section

> **Page:** ↪︎ Section
> **Type:** Component — Data Entry

---

## Overview

A content section container. Comes in two background variants: transparent (no background) and filled (light gray with border). Supports an empty state with an illustration, title, description, and an optional primary action button.

---

## Variants

| Variant | Background | Border |
|---|---|---|
| `No Background` | transparent | none |
| `With Background` | `#f1f0f0` | 1px solid `#e2e1e1` |

---

## Dimensions

| Property | Value |
|---|---|
| Corner radius (with bg) | 6px |
| Empty state illustration | 100px |
| Title font | 14px/26px semibold |
| Description font | 14px/26px regular |

---

## Colors

| Element | Color |
|---|---|
| Background (filled variant) | `#f1f0f0` |
| Border (filled variant) | `#e2e1e1` |
| Title text | `#242424` |
| Description text | `#625e5e` |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `color/neutral/100` | `#f1f0f0` | Filled section bg |
| `color/neutral/200` | `#e2e1e1` | Section border |
| `color/neutral/900-black` | `#242424` | Title text |
| `color/neutral/600` | `#625e5e` | Description text |

---

## CSS Custom Properties

```css
:root {
  --section-bg:          #f1f0f0;
  --section-border:      #e2e1e1;
  --section-title:       #242424;
  --section-description: #625e5e;
}
```

---

## CSS Classes

```css
/* ── Section wrapper ──────────────────────── */
.bos-section {
  display: flex;
  flex-direction: column;
  width: 100%;
  font-family: var(--font-family);
}

/* ── With background variant ──────────────── */
.bos-section--bg {
  background-color: var(--section-bg, #f1f0f0);
  border: 1px solid var(--section-border, #e2e1e1);
  border-radius: var(--corner-s, 6px);
}

/* ── Content area ─────────────────────────── */
.bos-section__content {
  width: 100%;
}

/* ── Empty state ──────────────────────────── */
.bos-section__empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 24px 16px;
  text-align: center;
}

.bos-section__illustration {
  width: 100px;
  height: 100px;
  flex-shrink: 0;
}

.bos-section__title {
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-semibold, 600);
  color: var(--section-title, #242424);
}

.bos-section__description {
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--section-description, #625e5e);
}
```

---

## HTML Usage

```html
<!-- Section — no background, with content -->
<div class="bos-section">
  <div class="bos-section__content">
    <!-- slot for any child content -->
  </div>
</div>

<!-- Section — with background, with content -->
<div class="bos-section bos-section--bg">
  <div class="bos-section__content">
    <!-- slot for any child content -->
  </div>
</div>

<!-- Section — empty state (no background) -->
<div class="bos-section">
  <div class="bos-section__empty">
    <img class="bos-section__illustration" src="empty-illustration.svg" alt="" />
    <p class="bos-section__title">ยังไม่มีข้อมูล</p>
    <p class="bos-section__description">ยังไม่มีรายการในระบบขณะนี้</p>
    <button class="bos-btn bos-btn--primary bos-btn--sm">เพิ่มรายการ</button>
  </div>
</div>

<!-- Section — empty state (with background) -->
<div class="bos-section bos-section--bg">
  <div class="bos-section__empty">
    <img class="bos-section__illustration" src="empty-illustration.svg" alt="" />
    <p class="bos-section__title">ไม่พบข้อมูล</p>
    <p class="bos-section__description">ลองปรับตัวกรองหรือค้นหาใหม่อีกครั้ง</p>
  </div>
</div>
```

---

## Usage Notes

- The Section component is a layout container — its size is determined by its content.
- The empty state illustration slot is 100px; use an SVG or PNG asset.
- The action button in the empty state is optional — use only when a primary action is available.
- Apply `bos-section--bg` to add the filled gray background and border; omit for a transparent container.
- Corner radius (6px) applies only to the `--bg` variant.
