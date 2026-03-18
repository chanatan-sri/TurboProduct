# Date Picker

> **Page:** ↪︎ Date picker
> **Type:** Component — Data Entry

---

## Overview

A calendar dropdown for selecting a single date or a date range. Rendered as a floating panel with a month/year header, day-of-week headers (Thai: อา จ อ พ พฤ ศ ส), and a 7×5 date grid. A date range picker variant shows two months side by side.

---

## Variants

| Variant | Width | Description |
|---|---|---|
| `Date picker` | 304px | Single date selection |
| `Date range picker` | 624px | Start + end date selection |

---

## Panel Dimensions

| Property | Value |
|---|---|
| Width | 304px (single) / 624px (range) |
| Padding | 12px |
| Corner radius | 6px |
| Background | `#fcfcfc` |
| Border | 1px solid `#e2e1e1` |
| Shadow | `0px 8px 16px 0px rgba(0,0,0,0.05)` |

---

## Header

| Element | Description |
|---|---|
| Prev/Next buttons | 32×32px outline buttons — border `#e2e1e1`, corner 8px |
| Month dropdown | Input with dropdown arrow, 112px wide |
| Year dropdown | Input with dropdown arrow, 80px wide |

---

## Day Cell States

| State | Background | Border | Text |
|---|---|---|---|
| `Default` | transparent | none | `#242424` |
| `Today` | `#f2f7fc` | 1.5px solid `#7baee8` | `#242424` |
| `Hover` | `#e8f2fb` | none | `#242424` |
| `Selected` | `#2d79c8` | none | `#fafafa` |
| `Range` (in range) | `#e8f2fb` | none | `#242424` |
| `Disabled` | transparent | none | `#adaaaa`, opacity 50% |

---

## Day Cell Dimensions

| Property | Value |
|---|---|
| Size | 40 × 40px |
| Corner radius | 6px |
| Font | 14px/26px regular |
| Row gap | 8px top margin |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `input/text/datepicker-text-default` | `#242424` | Date text |
| `input/text/datepicker-text-disabled` | `#adaaaa` | Disabled date text |
| `input/stroke/datepicker-border-default` | `#e2e1e1` | Panel border |
| `color/primary/50` | `#f2f7fc` | Today bg |
| `color/primary/300` | `#7baee8` | Today border |
| `color/primary/500☆` | `#2d79c8` | Selected bg |
| `color/primary/50` | `#e8f2fb` | Range / hover bg |

---

## CSS Custom Properties

```css
:root {
  --datepicker-bg:          #fcfcfc;
  --datepicker-border:      #e2e1e1;
  --datepicker-text:        #242424;
  --datepicker-text-disabled: #adaaaa;
  --datepicker-today-bg:    #f2f7fc;
  --datepicker-today-border:#7baee8;
  --datepicker-hover-bg:    #e8f2fb;
  --datepicker-selected-bg: #2d79c8;
  --datepicker-selected-text:#fafafa;
}
```

---

## CSS Classes

```css
/* ── Panel ───────────────────────────────────── */
.bos-datepicker {
  background-color: var(--datepicker-bg, #fcfcfc);
  border: 1px solid var(--datepicker-border, #e2e1e1);
  border-radius: var(--corner-s, 6px);
  padding: var(--space-l, 12px);
  box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.05);
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 304px;
  font-family: var(--font-family);
}

/* Range variant */
.bos-datepicker--range { width: 624px; }

/* ── Header ──────────────────────────────────── */
.bos-datepicker__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.bos-datepicker__nav {
  width: 32px;
  height: 32px;
  border: 1px solid var(--datepicker-border, #e2e1e1);
  border-radius: var(--corner-m, 8px);
  background: #fafafa;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  flex-shrink: 0;
}

.bos-datepicker__nav svg { width: 20px; height: 20px; }

.bos-datepicker__selection {
  display: flex;
  align-items: center;
  gap: 8px;
}

.bos-datepicker__dropdown {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 2px 12px;
  background: var(--input-bg, #fafafa);
  border: 1px solid var(--datepicker-border, #e2e1e1);
  border-radius: var(--corner-s, 6px);
  font-size: var(--text-body-size);         /* 14px */
  color: var(--datepicker-text, #242424);
  cursor: pointer;
}

.bos-datepicker__dropdown--month { width: 112px; }
.bos-datepicker__dropdown--year  { width: 80px; }

/* ── Day headers ─────────────────────────────── */
.bos-datepicker__thead {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding-top: 8px;
}

.bos-datepicker__th {
  width: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: var(--text-body-size);
  color: var(--datepicker-text, #242424);
  text-align: center;
}

/* ── Day grid ────────────────────────────────── */
.bos-datepicker__tbody {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 280px;
}

.bos-datepicker__row {
  display: flex;
  align-items: center;
  width: 100%;
  margin-top: 8px;
}

/* ── Day cell ────────────────────────────────── */
.bos-datepicker__day {
  width: 40px;
  height: 40px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: var(--corner-s, 6px);
  font-size: var(--text-body-size);
  font-weight: var(--font-weight-regular);
  color: var(--datepicker-text, #242424);
  cursor: pointer;
  text-align: center;
}

.bos-datepicker__day:hover {
  background-color: var(--datepicker-hover-bg, #e8f2fb);
}

.bos-datepicker__day--today {
  background-color: var(--datepicker-today-bg, #f2f7fc);
  border: 1.5px solid var(--datepicker-today-border, #7baee8);
}

.bos-datepicker__day--selected {
  background-color: var(--datepicker-selected-bg, #2d79c8);
  color: var(--datepicker-selected-text, #fafafa);
}

.bos-datepicker__day--in-range {
  background-color: var(--datepicker-hover-bg, #e8f2fb);
  border-radius: 0;
}

.bos-datepicker__day--disabled {
  opacity: 0.5;
  color: var(--datepicker-text-disabled, #adaaaa);
  cursor: not-allowed;
  pointer-events: none;
}
```

---

## HTML Usage

```html
<!-- Date picker panel -->
<div class="bos-datepicker">
  <!-- Header -->
  <div class="bos-datepicker__header">
    <button class="bos-datepicker__nav">
      <svg width="20" height="20"><!-- arrow left --></svg>
    </button>
    <div class="bos-datepicker__selection">
      <div class="bos-datepicker__dropdown bos-datepicker__dropdown--month">
        <span>พฤศจิกายน</span>
        <svg width="16" height="16"><!-- chevron down --></svg>
      </div>
      <div class="bos-datepicker__dropdown bos-datepicker__dropdown--year">
        <span>2568</span>
        <svg width="16" height="16"><!-- chevron down --></svg>
      </div>
    </div>
    <button class="bos-datepicker__nav">
      <svg width="20" height="20"><!-- arrow right --></svg>
    </button>
  </div>

  <!-- Day headers -->
  <div class="bos-datepicker__thead">
    <div class="bos-datepicker__th">อา</div>
    <div class="bos-datepicker__th">จ</div>
    <div class="bos-datepicker__th">อ</div>
    <div class="bos-datepicker__th">พ</div>
    <div class="bos-datepicker__th">พฤ</div>
    <div class="bos-datepicker__th">ศ</div>
    <div class="bos-datepicker__th">ส</div>
  </div>

  <!-- Day grid -->
  <div class="bos-datepicker__tbody">
    <div class="bos-datepicker__row">
      <div class="bos-datepicker__day">1</div>
      <div class="bos-datepicker__day">2</div>
      <div class="bos-datepicker__day bos-datepicker__day--today">3</div>
      <div class="bos-datepicker__day bos-datepicker__day--selected">4</div>
      <div class="bos-datepicker__day">5</div>
      <div class="bos-datepicker__day">6</div>
      <div class="bos-datepicker__day">7</div>
    </div>
    <!-- ... more rows -->
    <div class="bos-datepicker__row">
      <div class="bos-datepicker__day bos-datepicker__day--disabled">1</div>
      <div class="bos-datepicker__day bos-datepicker__day--disabled">2</div>
      <!-- out-of-month dates are disabled -->
    </div>
  </div>
</div>
```

---

## Usage Notes

- The date picker panel is a **floating dropdown** — position it absolutely below the trigger input.
- Day headers use Thai abbreviated day names: **อา จ อ พ พฤ ศ ส**.
- Out-of-month dates are shown at 50% opacity with `disabled` styling.
- The **Today** date has a light blue background and a thicker (1.5px) blue border.
- For the date range picker, highlight the range between start and end dates with `bos-datepicker__day--in-range`.
- Panel uses Thai Buddhist Era years (พ.ศ.) — year display is +543 from Gregorian.
