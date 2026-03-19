# Card

> **Page:** ↪︎ Card
> **Type:** Component — Data Entry

---

## Overview

Cards are surface containers with a colored header bar. The header supports a title, optional status badge, verify badge, and action controls (primary button, secondary button, or segmented tabs). The body area below is a white content region.

---

## Card Header Variants

| Variant | Header Background | Title Color |
|---|---|---|
| `Primary` | `color/primary/50` ≈ `#e8f2fb` | `#025fac` |
| `Secondary` | `color/neutral/50` ≈ `#f1f0f0` | `#242424` |
| `Tertiary` | `#f2f7fc` | `#025fac` |

---

## Header Dimensions

| Property | Value |
|---|---|
| Height | 40px |
| Padding | 16px horizontal, 4px vertical |
| Corner radius (top) | 6px |
| Border bottom | 1px solid `#feecf0` |
| Title font | 14px/26px semibold `#025fac` |

---

## Header Action Controls

| Control | Description |
|---|---|
| `Status badge` | Green pill badge (bg `#00a848`, text `#fafafa`) |
| `Verify badge` | Check icon + semibold green text `#008a2b` |
| `Primary button` | Outline, border `#2d79c8`, text `#2d79c8` |
| `Secondary button` | Outline, border `#e2e1e1`, text `#242424` |
| `Segmented group` | Tab strip — active: bg `#d2e4f8`, border `#afcff3`, text `#025fac`; inactive: transparent |

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `card/background/header-tertiary` | `color/primary/50` | `#f2f7fc` | Tertiary header bg |
| `card/text/heading-tertiary` | `color/primary/700` | `#025fac` | Header title |
| `tabs/segment/background/active` | `color/primary/100` | `#d2e4f8` | Active tab bg |
| `tabs/segment/text/active` | `color/primary/800` | `#025fac` | Active tab text |
| `tabs/segment/text/default` | `color/neutral/900-black` | `#242424` | Inactive tab text |

---

## CSS Custom Properties

```css
:root {
  --card-header-bg-tertiary: #f2f7fc;
  --card-header-border:      #feecf0;
  --card-title:              #025fac;
  --card-body-bg:            #ffffff;
  --card-border:             #e2e1e1;
  --card-segment-active-bg:  #d2e4f8;
  --card-segment-active-text:#025fac;
}
```

---

## CSS Classes

```css
/* ── Card wrapper ────────────────────────────── */
.bos-card {
  border: 1px solid var(--card-border, #e2e1e1);
  border-radius: var(--corner-s, 6px);
  overflow: hidden;
  font-family: var(--font-family);
}

/* ── Card header ─────────────────────────────── */
.bos-card__header {
  display: flex;
  align-items: center;
  gap: 8px;
  height: 40px;
  padding: 4px 16px;
  background-color: var(--card-header-bg-tertiary, #f2f7fc);
  border-bottom: 1px solid var(--card-header-border, #feecf0);
  border-radius: var(--corner-s, 6px) var(--corner-s, 6px) 0 0;
}

/* Header variants */
.bos-card__header--primary   { background-color: #e8f2fb; }
.bos-card__header--secondary { background-color: #f1f0f0; border-bottom-color: #e2e1e1; }
.bos-card__header--tertiary  { background-color: #f2f7fc; }

/* ── Header title ────────────────────────────── */
.bos-card__title {
  flex: 1;
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-semibold, 600);
  color: var(--card-title, #025fac);
  white-space: nowrap;
}

.bos-card__header--secondary .bos-card__title { color: #242424; }

/* ── Card body ───────────────────────────────── */
.bos-card__body {
  background-color: var(--card-body-bg, #ffffff);
  padding: var(--space-l, 12px);
}

/* ── Verify badge ────────────────────────────── */
.bos-card__verify {
  display: inline-flex;
  align-items: center;
  gap: 2px;
  font-size: var(--text-body-size);
  font-weight: var(--font-weight-semibold, 600);
  color: #008a2b;
}

.bos-card__verify svg {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

/* ── Segmented group ─────────────────────────── */
.bos-card__segment {
  display: flex;
  align-items: center;
  padding: 2px;
  border-radius: var(--corner-s, 6px);
  background-color: rgba(0, 0, 0, 0.05);
}

.bos-card__segment-item {
  padding: 0 7px;
  height: 28px;
  display: flex;
  align-items: center;
  border-radius: var(--corner-xs, 4px);
  font-size: var(--text-body-size);
  color: var(--card-segment-active-text, #242424);
  cursor: pointer;
  white-space: nowrap;
}

.bos-card__segment-item--active {
  background-color: var(--card-segment-active-bg, #d2e4f8);
  border: 1px solid #afcff3;
  color: var(--card-segment-active-text, #025fac);
  box-shadow: 0px 1px 2px rgba(0, 0, 0, 0.05);
}
```

---

## HTML Usage

```html
<!-- Card with tertiary header + verify badge + primary button -->
<div class="bos-card">
  <div class="bos-card__header bos-card__header--tertiary">
    <span class="bos-card__title">Card Title</span>

    <!-- Verify badge (optional) -->
    <div class="bos-card__verify">
      <svg width="20" height="20"><!-- verify icon --></svg>
      Verified
    </div>

    <!-- Status badge (optional) -->
    <div class="bos-badge-count">Active</div>

    <!-- Secondary button (optional) -->
    <button class="bos-btn bos-btn--outline-secondary">ยกเลิก</button>

    <!-- Primary button (optional) -->
    <button class="bos-btn bos-btn--outline-primary">บันทึก</button>
  </div>

  <div class="bos-card__body">
    <!-- Content here -->
  </div>
</div>

<!-- Card with segmented group in header -->
<div class="bos-card">
  <div class="bos-card__header bos-card__header--tertiary">
    <span class="bos-card__title">Overview</span>
    <div class="bos-card__segment">
      <span class="bos-card__segment-item bos-card__segment-item--active">Day</span>
      <span class="bos-card__segment-item">Week</span>
      <span class="bos-card__segment-item">Month</span>
    </div>
  </div>
  <div class="bos-card__body">
    <!-- Content here -->
  </div>
</div>
```

---

## Usage Notes

- Card header height is fixed at **40px** — do not change.
- Use `Tertiary` (light blue) header for most cards; `Secondary` (gray) for neutral/secondary content sections.
- Only one action type in the header at a time — either buttons, segmented group, or badges.
- The body area has no enforced height — it expands with content.
