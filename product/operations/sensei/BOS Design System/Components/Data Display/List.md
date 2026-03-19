# List

> **Page:** ↪︎ List
> **Type:** Component — Data Display

---

## Overview

List items for displaying rows of content. Supports two sizes, three states, optional background, and optional left/right icon slots with an optional description line.

---

## Sizes

| Size | Height | Icon Size |
|---|---|---|
| Default | 32px | 20 × 20px |
| Small | 28px | 20 × 20px |

---

## States

| State | Background |
|---|---|
| Default | Transparent (no bg) or `#f1f0f0` (with bg) |
| Hover | `#f1f0f0` |
| Disabled | Opacity reduced, no interaction |

---

## Background Variant

| Variant | Description |
|---|---|
| `Background=No` | Transparent row — used in dropdowns, menus |
| `Background=Yes` | Light gray bg `#f1f0f0` — used in filled list containers |

---

## Anatomy

| Slot | Description |
|---|---|
| Left slot | Optional 20px icon on the left |
| Content | Title (required) + Description (optional) |
| Right slot | Optional 20px icon on the right |

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `text-default` | `color/neutral/900-black` | `#242424` | Title text |
| `text-subtle` (description) | `color/neutral/600` | `#625e5e` | Description text |
| `icon-default` | `color/neutral/600` | `#625e5e` | Icon color |
| `background-hover` | `color/neutral/100` | `#f1f0f0` | Hover state bg |
| `background-yes` | `color/neutral/100` | `#f1f0f0` | Background variant fill |

---

## CSS Custom Properties

```css
:root {
  --list-text:         #242424;   /* color/neutral/900-black */
  --list-text-desc:    #625e5e;   /* color/neutral/600 */
  --list-icon:         #625e5e;   /* color/neutral/600 */
  --list-hover-bg:     #f1f0f0;
  --list-bg:           #f1f0f0;
}
```

---

## CSS Classes

```css
/* ── List item ──────────────────────────────────── */
.bos-list-item {
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding: 2px 4px 2px 8px;
  border-radius: var(--corner-xs, 4px);
  width: 208px;
  cursor: pointer;
  box-sizing: border-box;
}

/* Sizes */
.bos-list-item--default { min-height: 32px; }
.bos-list-item--sm      { min-height: 28px; }

/* Background variant */
.bos-list-item--bg { background-color: var(--list-bg, #f1f0f0); }

/* States */
.bos-list-item:hover { background-color: var(--list-hover-bg, #f1f0f0); }

.bos-list-item--disabled {
  opacity: 0.4;
  cursor: not-allowed;
  pointer-events: none;
}

/* ── Slots ──────────────────────────────────────── */
.bos-list-item__left,
.bos-list-item__right {
  display: flex;
  align-items: center;
  padding: var(--space-xs, 4px);
  flex-shrink: 0;
}

.bos-list-item__left svg,
.bos-list-item__right svg {
  width: 20px;
  height: 20px;
  color: var(--list-icon, #625e5e);
}

/* ── Content ─────────────────────────────────────── */
.bos-list-item__content {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-width: 0;
}

.bos-list-item__title {
  font-family: var(--font-family);
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--list-text, #242424);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.bos-list-item__desc {
  font-family: var(--font-family);
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  font-weight: var(--font-weight-regular);
  color: var(--list-text-desc, #625e5e);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

---

## HTML Usage

```html
<!-- Default — no background, no description -->
<div class="bos-list-item bos-list-item--default">
  <div class="bos-list-item__left">
    <svg width="20" height="20"><!-- icon --></svg>
  </div>
  <div class="bos-list-item__content">
    <span class="bos-list-item__title">Title</span>
  </div>
  <div class="bos-list-item__right">
    <svg width="20" height="20"><!-- info icon --></svg>
  </div>
</div>

<!-- With description -->
<div class="bos-list-item bos-list-item--default">
  <div class="bos-list-item__left">
    <svg width="20" height="20"><!-- icon --></svg>
  </div>
  <div class="bos-list-item__content">
    <span class="bos-list-item__title">Title</span>
    <span class="bos-list-item__desc">Description</span>
  </div>
</div>

<!-- With background — small -->
<div class="bos-list-item bos-list-item--sm bos-list-item--bg">
  <div class="bos-list-item__content">
    <span class="bos-list-item__title">Title</span>
  </div>
</div>

<!-- Disabled -->
<div class="bos-list-item bos-list-item--default bos-list-item--disabled">
  <div class="bos-list-item__content">
    <span class="bos-list-item__title">Disabled Item</span>
  </div>
</div>
```

---

## Usage Notes

- Use `Background=No` inside dropdowns, command menus, or overlays.
- Use `Background=Yes` inside bordered list containers or card bodies.
- Left and right slots are optional — omit the slot div if unused.
- Width of 208px is the component default; set a custom width as needed in context.
