# Tooltip

> **Page:** ↪︎ Tooltip
> **Type:** Component — Data Display

---

## Overview

Tooltips display brief contextual information on hover or focus. They appear as dark floating bubbles with directional arrows in 8 positions, and support an optional image variant for richer content.

---

## Position Variants

| Variant | Arrow direction |
|---|---|
| `Top` | Arrow points down — tooltip above target |
| `Top-left` | Arrow at bottom-left |
| `Top-right` | Arrow at bottom-right |
| `Bottom` | Arrow points up — tooltip below target |
| `Bottom-left` | Arrow at top-left |
| `Bottom-right` | Arrow at top-right |
| `Left` | Arrow points right — tooltip to the left |
| `Right` | Arrow points left — tooltip to the right |

---

## Dimensions

| Property | Value |
|---|---|
| Min-width | 48px |
| Max-width | 200px |
| Padding | 8px horizontal, 4px vertical |
| Corner radius | 6px |
| Arrow size | 12 × 7px |
| Shadow | `0px 3px 3px 0px rgba(0,0,0,0.05)` |

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `background-default` | `color/Neutral/900-black` | `#242424` | Tooltip body bg |
| `text-default` | `color/Neutral/0-white` | `#fafafa` | Tooltip text |

---

## CSS Custom Properties

```css
:root {
  --tooltip-bg:   #242424;   /* color/Neutral/900-black */
  --tooltip-text: #fafafa;   /* color/Neutral/0-white */
}
```

---

## CSS Classes

```css
/* ── Tooltip wrapper ────────────────────────────── */
.bos-tooltip {
  position: absolute;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  align-items: center;
  filter: drop-shadow(0px 3px 3px rgba(0, 0, 0, 0.05));
  pointer-events: none;
}

/* ── Body ───────────────────────────────────────── */
.bos-tooltip__body {
  background-color: var(--tooltip-bg, #242424);
  color: var(--tooltip-text, #fafafa);
  font-family: var(--font-family);
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  padding: var(--space-xs, 4px) var(--space-m, 8px);
  border-radius: 6px;
  min-width: 48px;
  max-width: 200px;
  white-space: nowrap;
  text-align: center;
}

/* ── Arrow ──────────────────────────────────────── */
.bos-tooltip__arrow {
  width: 12px;
  height: 7px;
  flex-shrink: 0;
}

/* ── Position variants ──────────────────────────── */

/* Top — tooltip above, arrow points down */
.bos-tooltip--top {
  flex-direction: column;
  bottom: calc(100% + 4px);
  left: 50%;
  transform: translateX(-50%);
}
.bos-tooltip--top .bos-tooltip__arrow {
  border-left: 6px solid transparent;
  border-right: 6px solid transparent;
  border-top: 7px solid var(--tooltip-bg, #242424);
  width: 0; height: 0;
  background: none;
}

/* Bottom — tooltip below, arrow points up */
.bos-tooltip--bottom {
  flex-direction: column-reverse;
  top: calc(100% + 4px);
  left: 50%;
  transform: translateX(-50%);
}
.bos-tooltip--bottom .bos-tooltip__arrow {
  border-left: 6px solid transparent;
  border-right: 6px solid transparent;
  border-bottom: 7px solid var(--tooltip-bg, #242424);
  width: 0; height: 0;
  background: none;
}

/* Left — tooltip to the left */
.bos-tooltip--left {
  flex-direction: row;
  right: calc(100% + 4px);
  top: 50%;
  transform: translateY(-50%);
}
.bos-tooltip--left .bos-tooltip__arrow {
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  border-left: 7px solid var(--tooltip-bg, #242424);
  width: 0; height: 0;
  background: none;
}

/* Right — tooltip to the right */
.bos-tooltip--right {
  flex-direction: row-reverse;
  left: calc(100% + 4px);
  top: 50%;
  transform: translateY(-50%);
}
.bos-tooltip--right .bos-tooltip__arrow {
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  border-right: 7px solid var(--tooltip-bg, #242424);
  width: 0; height: 0;
  background: none;
}

/* Top-left / Top-right — offset arrow position */
.bos-tooltip--top-left  { left: 0; transform: none; }
.bos-tooltip--top-right { right: 0; left: auto; transform: none; }
.bos-tooltip--bottom-left  { left: 0; transform: none; }
.bos-tooltip--bottom-right { right: 0; left: auto; transform: none; }

/* ── Trigger wrapper ────────────────────────────── */
.bos-tooltip-trigger {
  position: relative;
  display: inline-flex;
}

.bos-tooltip-trigger .bos-tooltip {
  display: none;
}

.bos-tooltip-trigger:hover .bos-tooltip,
.bos-tooltip-trigger:focus-within .bos-tooltip {
  display: flex;
}

/* ── Tooltip with image (rich content) ──────────── */
.bos-tooltip--rich .bos-tooltip__body {
  max-width: 332px;
  white-space: normal;
  display: flex;
  flex-direction: column;
  gap: var(--space-m, 8px);
  padding: var(--space-m, 8px);
}

.bos-tooltip--rich .bos-tooltip__image {
  width: 100%;
  border-radius: var(--corner-xs, 4px);
  display: block;
}
```

---

## HTML Usage

```html
<!-- Tooltip Top (CSS hover) -->
<div class="bos-tooltip-trigger">
  <button class="bos-btn bos-btn--outline-secondary">Hover me</button>

  <div class="bos-tooltip bos-tooltip--top">
    <div class="bos-tooltip__body">Tooltip text</div>
    <div class="bos-tooltip__arrow"></div>
  </div>
</div>

<!-- Tooltip Bottom -->
<div class="bos-tooltip-trigger">
  <span>?</span>
  <div class="bos-tooltip bos-tooltip--bottom">
    <div class="bos-tooltip__arrow"></div>
    <div class="bos-tooltip__body">Help text shown below</div>
  </div>
</div>

<!-- Tooltip Left -->
<div class="bos-tooltip-trigger">
  <button aria-label="Info">ℹ</button>
  <div class="bos-tooltip bos-tooltip--left">
    <div class="bos-tooltip__body">Description on the left</div>
    <div class="bos-tooltip__arrow"></div>
  </div>
</div>

<!-- Rich tooltip with image (Top) -->
<div class="bos-tooltip-trigger">
  <button>Preview</button>
  <div class="bos-tooltip bos-tooltip--top bos-tooltip--rich">
    <div class="bos-tooltip__body">
      <img class="bos-tooltip__image" src="preview.jpg" alt="Preview" />
      <span>Caption text here</span>
    </div>
    <div class="bos-tooltip__arrow"></div>
  </div>
</div>
```

---

## Usage Notes

- Default tooltip is **text-only** with `max-width: 200px` and `white-space: nowrap`.
- **Rich tooltips** support images and multi-line text (`max-width: 332px`).
- Arrow position must match the tooltip direction — Top tooltip has a downward-pointing arrow.
- Tooltips are shown on hover/focus; use CSS `:hover` or JavaScript for controlled visibility.
- Do not put interactive elements (buttons, links) inside standard tooltips.
