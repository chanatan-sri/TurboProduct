# Selection

> **Page:** ↪︎ Selection
> **Type:** Component — Data Entry

---

## Overview

An outlined pill/rectangle button used for selecting a single option from a set. Similar to a Chip but with square corners (corner-m = 8px). Supports three sizes and two selection states. Outlined style only.

---

## Sizes

| Size | Height | Min-width |
|---|---|---|
| `Small` | 28px | — |
| `Default` | 32px | — |
| `Large` | 40px | — |

---

## States

| State | Background | Border | Text |
|---|---|---|---|
| `Deselected` | `#fafafa` | `#c7c5c5` | `#4a4646` |
| `Selected` | `#fafafa` | `#2d79c8` | `#025fac` |
| `Disabled` | `#fafafa` | `#c7c5c5` | `#adaaaa`, opacity 50% |

---

## Dimensions

| Property | Value |
|---|---|
| Corner radius | 8px |
| Horizontal padding | 12px |
| Font | 14px/26px regular |
| Border width | 1px |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `color/neutral/0-white` | `#fafafa` | Background |
| `color/neutral/250` | `#c7c5c5` | Deselected border |
| `color/primary/500☆` | `#2d79c8` | Selected border |
| `color/neutral/700` | `#4a4646` | Deselected text |
| `color/primary/700` | `#025fac` | Selected text |
| `color/neutral/300` | `#adaaaa` | Disabled text |

---

## CSS Custom Properties

```css
:root {
  --selection-bg:              #fafafa;
  --selection-border:          #c7c5c5;
  --selection-border-selected: #2d79c8;
  --selection-text:            #4a4646;
  --selection-text-selected:   #025fac;
  --selection-text-disabled:   #adaaaa;
}
```

---

## CSS Classes

```css
/* ── Base selection ───────────────────────── */
.bos-selection {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-xs, 4px);
  height: 32px;
  padding: 0 var(--space-l, 12px);
  border-radius: var(--corner-m, 8px);
  border: 1px solid var(--selection-border, #c7c5c5);
  background-color: var(--selection-bg, #fafafa);
  font-family: var(--font-family);
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--selection-text, #4a4646);
  cursor: pointer;
  white-space: nowrap;
}

/* ── Selected state ───────────────────────── */
.bos-selection--selected {
  border-color: var(--selection-border-selected, #2d79c8);
  color: var(--selection-text-selected, #025fac);
}

/* ── Sizes ────────────────────────────────── */
.bos-selection--sm { height: 28px; }
.bos-selection--lg { height: 40px; }

/* ── Disabled ─────────────────────────────── */
.bos-selection--disabled {
  opacity: 0.5;
  color: var(--selection-text-disabled, #adaaaa);
  cursor: not-allowed;
  pointer-events: none;
}
```

---

## HTML Usage

```html
<!-- Deselected (default) -->
<button class="bos-selection">ทั้งหมด</button>

<!-- Selected -->
<button class="bos-selection bos-selection--selected">อนุมัติ</button>

<!-- Small — deselected -->
<button class="bos-selection bos-selection--sm">ร่าง</button>

<!-- Small — selected -->
<button class="bos-selection bos-selection--sm bos-selection--selected">สำเร็จ</button>

<!-- Large — deselected -->
<button class="bos-selection bos-selection--lg">ประเภท</button>

<!-- Large — selected -->
<button class="bos-selection bos-selection--lg bos-selection--selected">ประเภท A</button>

<!-- Disabled -->
<button class="bos-selection bos-selection--disabled" disabled>ไม่พร้อมใช้</button>

<!-- Selection group -->
<div style="display: flex; gap: 8px; flex-wrap: wrap;">
  <button class="bos-selection bos-selection--selected">ทั้งหมด</button>
  <button class="bos-selection">รออนุมัติ</button>
  <button class="bos-selection">อนุมัติแล้ว</button>
  <button class="bos-selection">ปฏิเสธ</button>
</div>
```

---

## Usage Notes

- Selection uses `corner-m` (8px) — unlike Chip which is fully pill-shaped (9999px).
- Only one option in a group should carry `bos-selection--selected` at a time (radio-like behavior).
- Groups should be laid out in a flex row with 8px gap, allowing wrapping on narrow screens.
- No check icon is shown on selection (unlike Filled Chip selected state).
