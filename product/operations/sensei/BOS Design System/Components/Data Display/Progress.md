# Progress

> **Page:** ↪︎ Progress
> **Type:** Component — Data Display

---

## Overview

Progress indicators communicate ongoing operations. BOS provides two variants: a **linear bar** (determinate or animated) and a **circular spinner** (indeterminate loading).

> **Note:** Design tokens for this component are marked `*TBC*` in Figma — values below are sourced from the rendered component.

---

## Variants

| Variant | Description |
|---|---|
| `Linear` | Horizontal bar — track + fill |
| `Circular` | Spinning ring — indeterminate loading indicator |

---

## Linear Progress Bar

### Dimensions

| Property | Value |
|---|---|
| Height | 4px |
| Corner radius | 4px |
| Width | 100% of container |

### Colors

| Part | Hex | Description |
|---|---|---|
| Track | `#d7d7d7` | Background rail |
| Fill (solid) | `#5494db` | Determinate progress fill |
| Fill (animated) | Gradient `#fafafa` → `#5494db` | Indeterminate shimmer animation |

### CSS

```css
/* ── Linear — track ─────────────────────────────── */
.bos-progress {
  width: 100%;
  height: 4px;
  background-color: #d7d7d7;
  border-radius: 4px;
  overflow: hidden;
  position: relative;
}

/* ── Determinate fill ───────────────────────────── */
.bos-progress__fill {
  height: 4px;
  border-radius: 4px;
  background-color: #5494db;
  transition: width 0.3s ease;
}

/* ── Indeterminate (animated shimmer) ───────────── */
.bos-progress--indeterminate .bos-progress__fill {
  width: 40%;
  background: linear-gradient(
    90deg,
    #fafafa 0%,
    #d6e4f3 22%,
    #95bce7 61%,
    #75a8e1 80%,
    #5494db 100%
  );
  animation: bos-progress-shimmer 1.5s infinite linear;
  position: absolute;
  left: -40%;
}

@keyframes bos-progress-shimmer {
  0%   { left: -40%; }
  100% { left: 100%; }
}
```

### HTML

```html
<!-- Determinate — 60% complete -->
<div class="bos-progress">
  <div class="bos-progress__fill" style="width: 60%;"></div>
</div>

<!-- Indeterminate / loading -->
<div class="bos-progress bos-progress--indeterminate">
  <div class="bos-progress__fill"></div>
</div>
```

---

## Circular Progress (Spinner)

A 16px spinning ring used for inline loading states.

### Dimensions

| Property | Value |
|---|---|
| Size | 16 × 16px |
| Color | `#5494db` (primary blue) |
| Animation | Continuous rotation |

### CSS

```css
.bos-spinner {
  width: 16px;
  height: 16px;
  border: 2px solid #d7d7d7;
  border-top-color: #5494db;
  border-radius: 50%;
  animation: bos-spin 0.8s linear infinite;
  flex-shrink: 0;
}

@keyframes bos-spin {
  to { transform: rotate(360deg); }
}
```

### HTML

```html
<!-- Inline spinner -->
<div class="bos-spinner" role="status" aria-label="Loading..."></div>
```

---

## Usage Notes

- **Linear determinate**: set `width` as a percentage to reflect actual progress.
- **Linear indeterminate**: use the shimmer animation when progress amount is unknown.
- **Circular spinner**: use inline (next to text or inside buttons) for loading states.
- Keep height at **4px** for the linear bar — do not change.
- Circular spinner matches the scroll-bar thumb color (`#5494db`) for visual consistency.
