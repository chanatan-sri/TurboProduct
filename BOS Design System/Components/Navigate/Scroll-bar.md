# Scroll Bar

> **Page:** ↪︎ Scroll bar
> **Type:** Component — Navigate

---

## Overview

A minimal horizontal scroll bar used inside scrollable containers. Consists of a track (background) and a thumb (active indicator).

---

## Anatomy

| Part | Description |
|---|---|
| Track | Full-width background bar |
| Thumb | Active scroll position indicator |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `scroll-bar/background/default` | `#f1f0f0` | Track background |
| `scroll-bar/action/default` | `#5494db` | Thumb / active indicator |

---

## Dimensions

| Property | Value |
|---|---|
| Height | 4px |
| Corner radius | `var(--corner-xs, 4px)` |
| Thumb width (example) | 60px (variable based on content) |

---

## CSS Custom Properties

```css
:root {
  --scroll-bar-bg:    #f1f0f0;  /* scroll-bar/background/default */
  --scroll-bar-thumb: #5494db;  /* scroll-bar/action/default */
}
```

---

## CSS Classes

```css
/* Horizontal scroll bar track */
.bos-scrollbar {
  background-color: var(--scroll-bar-bg, #f1f0f0);
  height: 4px;
  border-radius: var(--corner-xs, 4px);
  overflow: hidden;
  position: relative;
  width: 100%;
}

/* Thumb */
.bos-scrollbar__thumb {
  background-color: var(--scroll-bar-thumb, #5494db);
  height: 4px;
  border-radius: var(--corner-xs, 4px);
  position: absolute;
  top: 0;
  left: 0;
  /* Width is set dynamically based on scroll ratio */
}
```

### Native Scrollbar Styling

```css
/* Override native browser scrollbar to match BOS style */
.bos-scroll-container::-webkit-scrollbar {
  height: 4px;
}

.bos-scroll-container::-webkit-scrollbar-track {
  background: var(--scroll-bar-bg, #f1f0f0);
  border-radius: var(--corner-xs, 4px);
}

.bos-scroll-container::-webkit-scrollbar-thumb {
  background: var(--scroll-bar-thumb, #5494db);
  border-radius: var(--corner-xs, 4px);
}
```

---

## HTML Usage

```html
<!-- Custom scroll bar (decorative / overlaid) -->
<div class="bos-scrollbar">
  <div class="bos-scrollbar__thumb" style="width: 30%;"></div>
</div>

<!-- Native scrollbar override -->
<div class="bos-scroll-container" style="overflow-x: auto;">
  <!-- Scrollable content here -->
</div>
```

---

## Usage Notes

- Height is always **4px** — do not change.
- The thumb width is proportional to the visible content ratio.
- Use the native CSS scrollbar override whenever possible to avoid JavaScript overhead.
