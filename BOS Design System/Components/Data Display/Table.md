# Table

> **Page:** ↪︎ Table
> **Type:** Component — Data Display

---

## Overview

A full-featured data table. Includes a toolbar, sortable/filterable column headers, and value cells supporting text, buttons, icons, tags, checkboxes, radio buttons, switches, and skeleton loading states.

---

## Anatomy

| Part | Description |
|---|---|
| Toolbar | Search, filter result count, and bulk action bar |
| Header row | Column headers — title + optional sort/filter icons |
| Value rows | Data cells — multiple content types per cell |

---

## Header Column Types

| Type | Alignment |
|---|---|
| `Left column` | Left-aligned content |
| `Center column` | Center-aligned content |
| `Right column` | Right-aligned content |

---

## Header States

| State | Background |
|---|---|
| `Default` | `#f1f0f0` (heading bg) |
| `Hover` | `#e2e1e1` (heading hover) |

---

## Value Cell Content Types

| Type | Description |
|---|---|
| `Text` | Plain text value |
| `Input` | Inline editable field |
| `Checkbox` | Select row checkbox |
| `Radio` | Select row radio |
| `Button` | Action button inside cell |
| `Tag` | Status tag |
| `Switch` | Toggle switch |
| `Icon` | Icon action |
| `Status` | Badge/Status indicator |
| `Skeleton` | Loading placeholder |

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `background-heading-default` | `color/neutral/100` | `#f1f0f0` | Header row bg |
| `background-heading-hover` | `color/neutral/200` | `#e2e1e1` | Header hover bg |
| `background-value-default` | `color/neutral/0-white` | `#ffffff` | Data row bg |
| `background-value-hover` | `color/primary/50` | `#e8f2fb` | Data row hover bg |
| `background-value-highlight` | `color/neutral/50` | `#fafafa` | Alternate/highlighted row |
| `icon-default` | `color/neutral/600` | `#625e5e` | Default cell icon |
| `icon-disabled` | `color/neutral/300` | `#adaaaa` | Disabled cell icon |
| `icon-active` | `color/primary/500☆` | `#2d79c8` | Sort active icon |
| `text-heading-default` | `color/Neutral/900-black` | `#242424` | Column header text |
| `text-value-default` | `color/Neutral/900-black` | `#242424` | Cell value text |
| `text-sorting-active` | `color/primary/500☆` | `#2d79c8` | Active sort column text |
| `stroke-default` | `color/neutral/100` | `#e2e1e1` | Row and column dividers |

---

## CSS Custom Properties

```css
:root {
  --table-heading-bg:          #f1f0f0;
  --table-heading-hover-bg:    #e2e1e1;
  --table-value-bg:            #ffffff;
  --table-value-hover-bg:      #e8f2fb;
  --table-highlight-bg:        #fafafa;
  --table-text-heading:        #242424;
  --table-text-value:          #242424;
  --table-text-sort-active:    #2d79c8;
  --table-icon:                #625e5e;
  --table-icon-disabled:       #adaaaa;
  --table-icon-active:         #2d79c8;
  --table-stroke:              #e2e1e1;
}
```

---

## CSS Classes

```css
/* ── Table wrapper ──────────────────────────────── */
.bos-table {
  width: 100%;
  border-collapse: collapse;
  font-family: var(--font-family);
  font-size: var(--text-body-size);   /* 14px */
  line-height: var(--text-body-line-height);
  font-weight: var(--font-weight-regular);
}

/* ── Header cell ────────────────────────────────── */
.bos-table th,
.bos-table__th {
  background-color: var(--table-heading-bg, #f1f0f0);
  border-bottom: 1px solid var(--table-stroke, #e2e1e1);
  border-left: 1px solid var(--table-stroke, #e2e1e1);
  color: var(--table-text-heading, #242424);
  font-weight: var(--font-weight-semibold, 600);
  padding: 0 var(--space-m, 8px);
  height: 45px;
  min-width: 100px;
  max-width: 450px;
  white-space: nowrap;
  text-align: left;
  position: relative;
  user-select: none;
}

.bos-table th:first-child,
.bos-table__th:first-child {
  border-left: none;
}

.bos-table th:hover,
.bos-table__th:hover {
  background-color: var(--table-heading-hover-bg, #e2e1e1);
  cursor: pointer;
}

/* Header inner layout */
.bos-table__th-inner {
  display: flex;
  align-items: center;
  gap: var(--space-xs, 4px);
}

.bos-table__th-title { flex: 1; }

/* Sort icon */
.bos-table__sort-icon {
  width: 16px;
  height: 16px;
  color: var(--table-icon, #625e5e);
  flex-shrink: 0;
}

.bos-table__th--sort-active .bos-table__sort-icon,
.bos-table__th--sort-active .bos-table__th-title {
  color: var(--table-text-sort-active, #2d79c8);
}

/* Filter icon */
.bos-table__filter-icon {
  width: 16px;
  height: 16px;
  color: var(--table-icon, #625e5e);
  flex-shrink: 0;
}

/* ── Value row ───────────────────────────────────── */
.bos-table tr,
.bos-table__row {
  background-color: var(--table-value-bg, #ffffff);
  border-bottom: 1px solid var(--table-stroke, #e2e1e1);
}

.bos-table tr:hover,
.bos-table__row:hover {
  background-color: var(--table-value-hover-bg, #e8f2fb);
}

.bos-table tr.bos-table__row--highlight {
  background-color: var(--table-highlight-bg, #fafafa);
}

/* ── Value cell ─────────────────────────────────── */
.bos-table td,
.bos-table__td {
  padding: 0 var(--space-m, 8px);
  height: 39px;
  color: var(--table-text-value, #242424);
  border-left: 1px solid var(--table-stroke, #e2e1e1);
  vertical-align: middle;
  text-align: left;
}

.bos-table td:first-child,
.bos-table__td:first-child {
  border-left: none;
}

/* Disabled cell */
.bos-table td.bos-table__td--disabled {
  color: var(--table-icon-disabled, #adaaaa);
}

/* Alignment helpers */
.bos-table .bos-table__td--center { text-align: center; }
.bos-table .bos-table__td--right  { text-align: right; }

/* ── Toolbar ─────────────────────────────────────── */
.bos-table-toolbar {
  display: flex;
  align-items: center;
  gap: var(--space-m, 8px);
  padding: var(--space-m, 8px) var(--space-l, 12px);
  background-color: #ffffff;
  border-bottom: 1px solid #e2e1e1;
  min-height: 48px;
}

.bos-table-toolbar__search {
  flex: 1;
}

/* Skeleton cell */
.bos-table__skeleton {
  height: 12px;
  background-color: #e2e1e1;
  border-radius: 4px;
  animation: bos-skeleton-pulse 1.5s ease-in-out infinite;
}

@keyframes bos-skeleton-pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}
```

---

## HTML Usage

```html
<table class="bos-table">
  <thead>
    <tr>
      <th class="bos-table__th">
        <div class="bos-table__th-inner">
          <span class="bos-table__th-title">Column Title</span>
          <svg class="bos-table__filter-icon" width="16" height="16"><!-- filter --></svg>
          <svg class="bos-table__sort-icon" width="16" height="16"><!-- sort --></svg>
        </div>
      </th>
      <th class="bos-table__th bos-table__th--sort-active">
        <div class="bos-table__th-inner">
          <span class="bos-table__th-title">Active Sort</span>
          <svg class="bos-table__sort-icon" width="16" height="16"><!-- sort-asc --></svg>
        </div>
      </th>
      <th class="bos-table__th" style="text-align: right;">Amount</th>
    </tr>
  </thead>
  <tbody>
    <!-- Normal row -->
    <tr class="bos-table__row">
      <td class="bos-table__td">Text value</td>
      <td class="bos-table__td bos-table__td--center">
        <!-- Tag cell -->
        <span class="bos-tag bos-tag--blue">Active</span>
      </td>
      <td class="bos-table__td bos-table__td--right">฿ 10,000</td>
    </tr>

    <!-- Disabled row -->
    <tr class="bos-table__row">
      <td class="bos-table__td bos-table__td--disabled">Disabled text</td>
      <td class="bos-table__td"></td>
      <td class="bos-table__td bos-table__td--right bos-table__td--disabled">-</td>
    </tr>

    <!-- Skeleton row -->
    <tr class="bos-table__row">
      <td class="bos-table__td"><div class="bos-table__skeleton" style="width: 80%;"></div></td>
      <td class="bos-table__td"><div class="bos-table__skeleton" style="width: 60%;"></div></td>
      <td class="bos-table__td"><div class="bos-table__skeleton" style="width: 40%;"></div></td>
    </tr>
  </tbody>
</table>
```

---

## Usage Notes

- Header text is **semibold (600)**, value text is regular.
- Sort active column changes both icon and text to primary blue `#2d79c8`.
- Hover row uses primary-50 blue tint `#e8f2fb` — not gray.
- First column has no left border.
- Use the `bos-table-toolbar` bar above the table for search, filters, and bulk actions.
