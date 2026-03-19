# Slider

> **Page:** ↪︎ Slider
> **Type:** Component — Data Entry

---

## Overview

A range slider for selecting a numeric value along a track. The filled (active) portion of the track extends from the left edge to the thumb. Supports default and disabled states.

---

## Dimensions

| Property | Value |
|---|---|
| Track height | 4px |
| Track corner radius | 9999px (pill) |
| Thumb diameter | 18px |
| Thumb corner radius | 50% (circle) |

---

## Colors

| Element | State | Color |
|---|---|---|
| Track (filled / active) | Default | `#5494db` |
| Track (unfilled) | Default | `#f1f0f0` |
| Thumb | Default | `#2d79c8` |
| Track (filled) | Disabled | `#e2e1e1` |
| Track (unfilled) | Disabled | `#f1f0f0` |
| Thumb | Disabled | `#e2e1e1` |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `color/primary/400` | `#5494db` | Active track fill |
| `color/primary/500☆` | `#2d79c8` | Thumb |
| `color/neutral/100` | `#f1f0f0` | Inactive track |
| `color/neutral/200` | `#e2e1e1` | Disabled track / thumb |

---

## CSS Custom Properties

```css
:root {
  --slider-track-active:   #5494db;
  --slider-track-inactive: #f1f0f0;
  --slider-thumb:          #2d79c8;
  --slider-thumb-disabled: #e2e1e1;
}
```

---

## CSS Classes

```css
/* ── Wrapper ──────────────────────────────── */
.bos-slider {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
  font-family: var(--font-family);
}

/* ── Track container ──────────────────────── */
.bos-slider__track {
  position: relative;
  width: 100%;
  height: 4px;
  border-radius: 9999px;
  background-color: var(--slider-track-inactive, #f1f0f0);
}

/* ── Active fill ──────────────────────────── */
.bos-slider__fill {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  border-radius: 9999px;
  background-color: var(--slider-track-active, #5494db);
  /* width set inline via JS, e.g. style="width: 40%" */
}

/* ── Thumb ────────────────────────────────── */
.bos-slider__thumb {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background-color: var(--slider-thumb, #2d79c8);
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.15);
  cursor: grab;
  /* left set inline via JS, e.g. style="left: 40%" */
}

.bos-slider__thumb:active { cursor: grabbing; }

/* ── Native range input (accessible overlay) */
.bos-slider__input {
  position: absolute;
  inset: 0;
  width: 100%;
  opacity: 0;
  cursor: pointer;
  height: 18px;
  top: 50%;
  transform: translateY(-50%);
}

/* ── Value label ──────────────────────────── */
.bos-slider__value {
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  color: #242424;
  text-align: right;
}

/* ── Disabled ─────────────────────────────── */
.bos-slider--disabled {
  pointer-events: none;
}

.bos-slider--disabled .bos-slider__fill {
  background-color: var(--slider-thumb-disabled, #e2e1e1);
}

.bos-slider--disabled .bos-slider__thumb {
  background-color: var(--slider-thumb-disabled, #e2e1e1);
  cursor: not-allowed;
  box-shadow: none;
}
```

---

## HTML Usage

```html
<!-- Default slider at 40% -->
<div class="bos-slider">
  <div class="bos-slider__track">
    <div class="bos-slider__fill" style="width: 40%;"></div>
    <div class="bos-slider__thumb" style="left: 40%;"></div>
    <input class="bos-slider__input" type="range" min="0" max="100" value="40" />
  </div>
  <span class="bos-slider__value">40</span>
</div>

<!-- Slider at minimum (0%) -->
<div class="bos-slider">
  <div class="bos-slider__track">
    <div class="bos-slider__fill" style="width: 0%;"></div>
    <div class="bos-slider__thumb" style="left: 0%;"></div>
    <input class="bos-slider__input" type="range" min="0" max="100" value="0" />
  </div>
</div>

<!-- Slider at maximum (100%) -->
<div class="bos-slider">
  <div class="bos-slider__track">
    <div class="bos-slider__fill" style="width: 100%;"></div>
    <div class="bos-slider__thumb" style="left: 100%;"></div>
    <input class="bos-slider__input" type="range" min="0" max="100" value="100" />
  </div>
</div>

<!-- Disabled -->
<div class="bos-slider bos-slider--disabled">
  <div class="bos-slider__track">
    <div class="bos-slider__fill" style="width: 60%;"></div>
    <div class="bos-slider__thumb" style="left: 60%;"></div>
    <input class="bos-slider__input" type="range" min="0" max="100" value="60" disabled />
  </div>
</div>
```

---

## Usage Notes

- The fill width and thumb `left` position are set to the same percentage value (the current value as a fraction of the range).
- Use the hidden `<input type="range">` as the accessible control — the visual track and thumb are decorative.
- Track height is always 4px; do not increase it.
- Thumb diameter is 18px for all sizes.
