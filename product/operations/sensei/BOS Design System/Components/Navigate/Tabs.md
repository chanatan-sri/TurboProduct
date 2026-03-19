# Tabs

> **Page:** ↪︎ Tabs
> **Type:** Component — Navigate

---

## Overview

BOS provides three tab variants for different use cases:

| Variant | Description |
|---|---|
| **Tab Line** | Underline-style tab bar — horizontal list of text tabs with active indicator line |
| **Segmented** | Pill-group switcher — compact toggle between 2–4 options |
| **Tab Pills** | Individual rounded pill tabs — inline filtering or categorising |

---

## Design Tokens

### Tabs / Text

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `text-default` | `color/neutral/900-black` | `#242424` | Inactive tab |
| `text-hover` | `color/primary/400` | `#2d79c8` | Hovered tab |
| `text-active` | `color/primary/500☆` | `#2d79c8` | Active tab |
| `text-disabled` | `color/neutral/300` | `#adaaaa` | Disabled tab |

### Tabs / Icon

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `icon-default` | `color/neutral/600` | `#625e5e` | Default icon |
| `icon-hover` | `color/primary/400` | `#2d79c8` | Hovered icon |
| `icon-active` | `color/primary/500☆` | `#2d79c8` | Active icon |
| `icon-disabled` | `color/neutral/300` | `#adaaaa` | Disabled icon |

### Tabs / Stroke (Tab Line)

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `stroke-default` | `color/neutral/100` | `#e2e1e1` | Bottom separator line |
| `stroke-active` | `color/primary/500☆` | `#2d79c8` | Active tab indicator |
| `stroke-disabled` | `color/neutral/300` | `#adaaaa` | Disabled indicator |

---

## Variant 1 — Tab Line

### Visual Structure

```
Tab1   Tab2   Tab3   Tab4   Tab5
────                            ← 1px neutral/100 separator (full width)
```
Active tab has a **2px** bottom border in `color/primary/500☆`.

### CSS

```css
/* Tab line container */
.bos-tab-line {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

/* Tab items row */
.bos-tab-line__items {
  display: flex;
  gap: 16px;
  align-items: flex-end;
}

/* Single tab */
.bos-tab-line__tab {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  justify-content: flex-end;
  cursor: pointer;
  padding-bottom: 0;
}

/* Tab label */
.bos-tab-line__label {
  font-family: var(--font-family);
  font-size: var(--text-body-size);   /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--tabs-text-default, #242424);
  white-space: nowrap;
}

.bos-tab-line__tab:hover .bos-tab-line__label {
  color: #2d79c8;
}

.bos-tab-line__tab--active .bos-tab-line__label {
  color: #2d79c8;
}

.bos-tab-line__tab--disabled .bos-tab-line__label {
  color: #adaaaa;
  cursor: not-allowed;
  pointer-events: none;
}

/* Active indicator */
.bos-tab-line__indicator {
  height: 2px;
  width: 100%;
  background-color: #2d79c8;
  border-radius: 2px;
}

/* Bottom separator */
.bos-tab-line__separator {
  height: 1px;
  width: 100%;
  background-color: #e2e1e1;
}
```

### HTML — Tab Line

```html
<div class="bos-tab-line">
  <div class="bos-tab-line__items">
    <div class="bos-tab-line__tab bos-tab-line__tab--active">
      <span class="bos-tab-line__label">Tab1</span>
      <div class="bos-tab-line__indicator"></div>
    </div>
    <div class="bos-tab-line__tab">
      <span class="bos-tab-line__label">Tab2</span>
    </div>
    <div class="bos-tab-line__tab">
      <span class="bos-tab-line__label">Tab3</span>
    </div>
    <div class="bos-tab-line__tab bos-tab-line__tab--disabled">
      <span class="bos-tab-line__label">Tab4</span>
    </div>
  </div>
  <div class="bos-tab-line__separator"></div>
</div>
```

---

## Variant 2 — Segmented

A compact toggle bar, used for switching between 2–4 options (e.g., Daily / Weekly / Monthly).

### Sizes

| Size | Padding | Height |
|---|---|---|
| Small | 7px horizontal | ~30px |
| Medium | 11px H / 3px V | ~36px |
| Large | 11px H / 7px V | ~44px |

### Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `tabs/segment/background/active` | `#d2e4f8` | Active segment background |
| `tabs/segment/text/active` | `#025fac` | Active segment text |
| `tabs/segment/text/default` | `#242424` | Inactive segment text |
| Container bg | `rgba(0,0,0,0.05)` over `#fafafa` | Outer track |
| Active border | `#afcff3` (`color/primary/200`) | Active segment border |

### CSS

```css
/* Segmented container */
.bos-segmented {
  display: inline-flex;
  align-items: center;
  padding: 2px;
  border-radius: var(--corner-s, 6px);
  background: rgba(0, 0, 0, 0.05);
}

/* Segment item */
.bos-segmented__item {
  display: flex;
  align-items: center;
  gap: var(--space-m, 8px);
  padding: 0 11px;
  border-radius: var(--corner-xs, 4px);
  font-family: var(--font-family);
  font-size: var(--text-body-size);
  line-height: var(--text-body-line-height);
  font-weight: var(--font-weight-regular);
  color: #242424;
  cursor: pointer;
  background-color: transparent;
  border: 1px solid transparent;
  white-space: nowrap;
}

/* Size variants */
.bos-segmented--sm .bos-segmented__item { padding: 0 7px; }
.bos-segmented--md .bos-segmented__item { padding: 3px 11px; }
.bos-segmented--lg .bos-segmented__item { padding: 7px 11px; }

/* Active segment */
.bos-segmented__item--active {
  background-color: #d2e4f8;
  border-color: #afcff3;
  color: #025fac;
  box-shadow: var(--shadow-xs, 0px 1px 2px 0px rgba(0,0,0,0.05));
}
```

### HTML — Segmented

```html
<!-- Medium size -->
<div class="bos-segmented bos-segmented--md">
  <button class="bos-segmented__item bos-segmented__item--active">Daily</button>
  <button class="bos-segmented__item">Weekly</button>
  <button class="bos-segmented__item">Monthly</button>
</div>
```

---

## Variant 3 — Tab Pills

Rounded pill-shaped individual tab buttons for filtering or categorising.

### States

| State | Background | Border | Text |
|---|---|---|---|
| Inactive | `#fafafa` | `#e2e1e1` (1px) | `#242424` |
| Active | `#d2e4f8` | transparent | `#2d79c8` |
| Disabled | `#f1f0f0` | transparent | `#adaaaa` |

### Dimensions

| Property | Value |
|---|---|
| Height | 32px |
| Min-width | 64px |
| Padding | 12px horizontal |
| Corner | `var(--corner-full, 9999px)` |

### CSS

```css
/* Tab pill */
.bos-tab-pill {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-m, 8px);
  min-width: 64px;
  height: 32px;
  padding: 0 var(--space-l, 12px);
  border-radius: var(--corner-full, 9999px);
  border: 1px solid #e2e1e1;
  background-color: #fafafa;
  font-family: var(--font-family);
  font-size: var(--text-body-size);
  line-height: var(--text-body-line-height);
  font-weight: var(--font-weight-regular);
  color: #242424;
  cursor: pointer;
  white-space: nowrap;
}

.bos-tab-pill--active {
  background-color: #d2e4f8;
  border-color: transparent;
  color: #2d79c8;
}

.bos-tab-pill--disabled {
  background-color: #f1f0f0;
  border-color: transparent;
  color: #adaaaa;
  cursor: not-allowed;
  pointer-events: none;
}
```

### HTML — Tab Pills

```html
<div style="display: flex; gap: 8px;">
  <button class="bos-tab-pill">Tab pills</button>
  <button class="bos-tab-pill bos-tab-pill--active">Tab pills</button>
  <button class="bos-tab-pill bos-tab-pill--disabled" disabled>Tab pills</button>
</div>
```

---

## Usage Notes

- **Tab Line** — use for primary page-level navigation (e.g. switching between views).
- **Segmented** — use for compact 2–4 option toggles within a panel (e.g. chart time range).
- **Tab Pills** — use for filter chips or category selectors in list/table headers.
- All variants support optional icons alongside the label.
