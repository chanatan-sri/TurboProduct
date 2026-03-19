# Color Picker

> **Page:** ↪︎ Color picker
> **Type:** Component — Data Entry

---

## Overview

A color swatch selector. Each color is shown as a 36px filled circle with a text label beneath or beside it. Two layout orientations: **Horizontal** (swatch + label in a row) and **Vertical** (swatch above label). Selected swatches show an outlined ring around the circle.

---

## Layouts

| Layout | Description |
|---|---|
| `Horizontal` | Swatch and label side by side in a row |
| `Vertical` | Swatch above label, centered |

---

## Swatch States

| State | Appearance |
|---|---|
| `Unselected` | Solid filled circle, no border |
| `Selected` | Solid filled circle + outer ring/outline |
| `Disabled` | Reduced opacity (50%) |

---

## Dimensions

| Property | Value |
|---|---|
| Swatch outer (selected ring) | 40px diameter |
| Swatch circle | 36px diameter |
| Corner radius | 50% (circle) |
| Label font | 12px/22px regular |
| Label color | `#322f2f` |

---

## Available Colors

| Label | Fill Color |
|---|---|
| Yellow | `#ffc107` |
| Orange | `#ff6d00` |
| Brown | `#795548` |
| Red | `#f44336` |
| Pink | `#e91e63` |
| Purple | `#9c27b0` |
| Blue | `#2d79c8` |
| Light Blue | `#03a9f4` |
| Green | `#4caf50` |
| Black | `#242424` |
| Grey | `#9e9e9e` |
| White | `#fafafa` |
| Multicolors | conic-gradient (rainbow) |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `color/neutral/50` | `#f5f5f5` | Swatch group background |
| `color/neutral/800` | `#322f2f` | Label text |
| `color/primary/500☆` | `#2d79c8` | Selected ring color |

---

## CSS Custom Properties

```css
:root {
  --color-picker-label:        #322f2f;
  --color-picker-ring:         #2d79c8;
  --color-picker-group-bg:     #f5f5f5;
}
```

---

## CSS Classes

```css
/* ── Swatch group ─────────────────────────── */
.bos-color-picker {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  font-family: var(--font-family);
}

/* ── Horizontal layout ────────────────────── */
.bos-color-picker--horizontal {
  flex-direction: row;
  align-items: center;
}

/* ── Vertical layout ──────────────────────── */
.bos-color-picker--vertical {
  flex-direction: row;
  align-items: flex-start;
}

/* ── Single swatch item ───────────────────── */
.bos-color-picker__item {
  display: flex;
  align-items: center;
  gap: 6px;
  cursor: pointer;
}

.bos-color-picker--vertical .bos-color-picker__item {
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

/* ── Swatch ring container ────────────────── */
.bos-color-picker__swatch-wrap {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid transparent;
  flex-shrink: 0;
}

.bos-color-picker__item--selected .bos-color-picker__swatch-wrap {
  border-color: var(--color-picker-ring, #2d79c8);
}

/* ── Swatch circle ────────────────────────── */
.bos-color-picker__swatch {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  flex-shrink: 0;
}

/* ── Label ────────────────────────────────── */
.bos-color-picker__label {
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  font-weight: var(--font-weight-regular);
  color: var(--color-picker-label, #322f2f);
  white-space: nowrap;
}

/* ── Disabled ─────────────────────────────── */
.bos-color-picker__item--disabled {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}
```

---

## HTML Usage

```html
<!-- Horizontal layout -->
<div class="bos-color-picker bos-color-picker--horizontal">
  <!-- Unselected -->
  <div class="bos-color-picker__item">
    <div class="bos-color-picker__swatch-wrap">
      <div class="bos-color-picker__swatch" style="background-color: #ffc107;"></div>
    </div>
    <span class="bos-color-picker__label">Yellow</span>
  </div>

  <!-- Selected -->
  <div class="bos-color-picker__item bos-color-picker__item--selected">
    <div class="bos-color-picker__swatch-wrap">
      <div class="bos-color-picker__swatch" style="background-color: #2d79c8;"></div>
    </div>
    <span class="bos-color-picker__label">Blue</span>
  </div>

  <div class="bos-color-picker__item">
    <div class="bos-color-picker__swatch-wrap">
      <div class="bos-color-picker__swatch" style="background-color: #4caf50;"></div>
    </div>
    <span class="bos-color-picker__label">Green</span>
  </div>
</div>

<!-- Vertical layout -->
<div class="bos-color-picker bos-color-picker--vertical">
  <div class="bos-color-picker__item">
    <div class="bos-color-picker__swatch-wrap">
      <div class="bos-color-picker__swatch" style="background-color: #f44336;"></div>
    </div>
    <span class="bos-color-picker__label">Red</span>
  </div>

  <div class="bos-color-picker__item bos-color-picker__item--selected">
    <div class="bos-color-picker__swatch-wrap">
      <div class="bos-color-picker__swatch" style="background-color: #e91e63;"></div>
    </div>
    <span class="bos-color-picker__label">Pink</span>
  </div>

  <div class="bos-color-picker__item">
    <div class="bos-color-picker__swatch-wrap">
      <div class="bos-color-picker__swatch" style="background-color: #9c27b0;"></div>
    </div>
    <span class="bos-color-picker__label">Purple</span>
  </div>
</div>

<!-- Multicolors swatch -->
<div class="bos-color-picker__item">
  <div class="bos-color-picker__swatch-wrap">
    <div class="bos-color-picker__swatch" style="background: conic-gradient(red, yellow, lime, cyan, blue, magenta, red);"></div>
  </div>
  <span class="bos-color-picker__label">Multicolors</span>
</div>

<!-- White swatch (needs border to be visible) -->
<div class="bos-color-picker__item">
  <div class="bos-color-picker__swatch-wrap">
    <div class="bos-color-picker__swatch" style="background-color: #fafafa; border: 1px solid #e2e1e1;"></div>
  </div>
  <span class="bos-color-picker__label">White</span>
</div>
```

---

## Usage Notes

- Swatch circle is always 36px; the 40px ring container provides the 2px selection ring gap.
- The selection ring uses the primary color (`#2d79c8`); the ring is shown only when `--selected` modifier is applied.
- White swatches should have a `1px solid #e2e1e1` border so they are visible against light backgrounds.
- Labels are in Thai for this design system context (e.g. "เหลือง", "แดง", "น้ำเงิน").
- Only one swatch should be selected at a time (radio-like behavior).
