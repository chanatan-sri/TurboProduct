# Paginate

> **Page:** ↪︎ Paginate
> **Type:** Component — Navigate

---

## Overview

Pagination allows users to navigate across multiple pages of content. It includes previous/next controls, page number items, ellipsis for collapsed ranges, a total count label, and an optional per-page selector.

---

## Anatomy

| Part | Description |
|---|---|
| Total text | "Total N items" — summary label |
| Prev button | Left arrow — disabled on first page |
| Page items | Numbered buttons |
| Ellipsis | `•••` — collapsed page range |
| Next button | Right arrow |
| Per-page selector | "Show [10 ▼] /100" dropdown |

---

## States

### Page Item

| State | Background | Border | Text |
|---|---|---|---|
| Default | `#fafafa` | `#e2e1e1` | `#242424` |
| Active | `#fafafa` | `#2d79c8` | `#2d79c8` |
| Hover | `#f1f0f0` | `#e2e1e1` | `#242424` |
| Disabled | `#f1f0f0` | `#e2e1e1` | `#adaaaa` |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `pagination/background/default` | `#fafafa` | Default page item background |
| `pagination/background/active` | `#fafafa` | Active page item background |
| `pagination/background/disabled` | `#f1f0f0` | Disabled prev/next background |
| `pagination/stroke/default` | `#e2e1e1` | Default item border |
| `pagination/stroke/active` | `#2d79c8` | Active item border |
| `pagination/stroke/disabled` | `#e2e1e1` | Disabled item border |
| `pagination/text/default` | `#242424` | Default page number text |
| `pagination/text/active` | `#2d79c8` | Active page number text |
| `pagination/text/elipsis-inactive` | `#adaaaa` | Ellipsis `•••` color |

---

## Dimensions

| Property | Value |
|---|---|
| Page item size | 32 × 32px |
| Prev/Next size | 32 × 32px |
| Item corner radius | 6px |
| Gap between items | 8px |
| Gap between groups | 16px |
| Font | Sarabun 14px / 26px regular |

---

## CSS Custom Properties

```css
:root {
  --pagination-bg-default:   #fafafa;
  --pagination-bg-active:    #fafafa;
  --pagination-bg-disabled:  #f1f0f0;
  --pagination-border:       #e2e1e1;
  --pagination-border-active: #2d79c8;
  --pagination-text:         #242424;
  --pagination-text-active:  #2d79c8;
  --pagination-text-ellipsis: #adaaaa;
}
```

---

## CSS Classes

```css
/* Pagination container */
.bos-pagination {
  display: flex;
  align-items: center;
  gap: 16px;
  font-family: var(--font-family);
  font-size: var(--text-body-size);   /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
}

/* Total text label */
.bos-pagination__total {
  color: var(--pagination-text, #242424);
  white-space: nowrap;
  padding-right: 8px;
}

/* Page items row */
.bos-pagination__items {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* Individual page item */
.bos-pagination__item {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border: 1px solid var(--pagination-border, #e2e1e1);
  background-color: var(--pagination-bg-default, #fafafa);
  border-radius: 6px;
  color: var(--pagination-text, #242424);
  font-family: var(--font-family);
  font-size: var(--text-body-size);
  cursor: pointer;
  text-align: center;
  user-select: none;
}

.bos-pagination__item:hover {
  background-color: #f1f0f0;
}

.bos-pagination__item--active {
  border-color: var(--pagination-border-active, #2d79c8);
  color: var(--pagination-text-active, #2d79c8);
  cursor: default;
}

.bos-pagination__item--disabled {
  background-color: var(--pagination-bg-disabled, #f1f0f0);
  border-color: var(--pagination-border, #e2e1e1);
  color: #adaaaa;
  cursor: not-allowed;
  pointer-events: none;
}

/* Ellipsis */
.bos-pagination__ellipsis {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--pagination-text-ellipsis, #adaaaa);
  border-radius: 6px;
}

/* Per-page options */
.bos-pagination__options {
  display: flex;
  align-items: center;
  gap: 4px;
  color: var(--pagination-text, #242424);
}

.bos-pagination__per-page {
  width: 70px;
  border: 1px solid #e2e1e1;
  border-radius: var(--corner-s, 6px);
  background: #fafafa;
  padding: 2px 12px;
  font-family: var(--font-family);
  font-size: var(--text-body-size);
  color: #625e5e;
}
```

---

## HTML Usage

```html
<div class="bos-pagination">
  <!-- Total label -->
  <span class="bos-pagination__total">Total 85 items</span>

  <!-- Page items -->
  <div class="bos-pagination__items">
    <!-- Prev (disabled) -->
    <button class="bos-pagination__item bos-pagination__item--disabled" aria-label="Previous">‹</button>

    <!-- Page numbers -->
    <button class="bos-pagination__item bos-pagination__item--active" aria-current="page">1</button>
    <button class="bos-pagination__item">2</button>
    <button class="bos-pagination__item">3</button>
    <button class="bos-pagination__item">4</button>
    <button class="bos-pagination__item">5</button>
    <button class="bos-pagination__item">6</button>

    <!-- Ellipsis -->
    <span class="bos-pagination__ellipsis">•••</span>

    <button class="bos-pagination__item">50</button>

    <!-- Next -->
    <button class="bos-pagination__item" aria-label="Next">›</button>
  </div>

  <!-- Per-page selector -->
  <div class="bos-pagination__options">
    <span>Show</span>
    <select class="bos-pagination__per-page">
      <option>10</option>
      <option>20</option>
      <option>50</option>
    </select>
    <span>/100</span>
  </div>
</div>
```

---

## Usage Notes

- The **active** page item uses a blue border, not a filled background.
- The **Prev** button is always disabled on page 1; the **Next** button is disabled on the last page.
- The **total text** and **per-page selector** are optional — hide if not needed.
- Always include `aria-current="page"` on the active item and `aria-label` on Prev/Next.
