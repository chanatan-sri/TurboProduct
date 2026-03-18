# Side Navigation Bar

> **Page:** ↪︎ Side navigation bar
> **Type:** Component — Navigate

---

## Overview

The side navigation bar is the primary vertical navigation. It supports:
- **Full** expanded mode (216px) with text labels, section titles, badge counts, and sub-menus
- **Collapsed** mode (~52px) with icon-only items and tooltip on hover
- **Section sidebar** — a step-list or list sidebar variant for secondary navigation
- **Right sidebar** — panel with optional scrollbar

---

## Variants

| Variant | Width | Shows |
|---|---|---|
| `Side bar-full` | 216px | Icon + label + badge + right arrow |
| `Side bar-collapse` | 52px | Icon only (tooltip on hover) |
| `side navigation bar (Expand=Yes)` | 240px | Full with sub-items |
| `side navigation bar (Expand=No)` | 52px | Collapsed icons |

---

## Design Tokens

### Background

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `background-default` | `color/neutral/0-white` | `#ffffff` | Main sidebar bg |
| `background-default` (alt) | `color/neutral/50` | `#fafafa` | Secondary bg level |
| `background-active` | `color/neutral/50` | `#fafafa` | Active item bg |
| `background-active-hover` | `color/neutral/50` | `#fafafa` | Active + hovered bg |
| `background-hover` | `color/neutral/50` | `#f1f0f0` | Hovered item bg |

### Icon

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `icon-default` | `color/neutral/600` | `#625e5e` | Default icon |
| `icon-default-subtle` | `color/neutral/600` | `#625e5e` | Secondary icon |
| `icon-hover` | `color/neutral/600` | `#625e5e` | Hovered icon |
| `icon-active` | `color/primary/500☆` | (primary brand) | Active item icon |

### Text

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `text-default` | `color/neutral/900-black` | `#242424` | Nav item label |
| `text-default-subtle` | `color/neutral/600` | `#625e5e` | Section title (12px) |
| `text-hover` | `color/neutral/900-black` | `#242424` | Hovered label |
| `text-active` | `color/neutral/900-black` | `#242424` | Active item label |

### Divider

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `divider-default` | `color/neutral/100` | `#e2e1e1` | Sidebar border, section divider |

---

## Dimensions

| Property | Value |
|---|---|
| Full sidebar width | 216px |
| Collapsed sidebar width | 52px |
| Nav item width (full) | 200px |
| Nav item height | 42px |
| Nav item padding | 8px (`var(--space-m)`) |
| Nav item corner radius | 4px (`var(--corner-xs)`) |
| Section title font | Sarabun 12px / 22px regular |
| Nav label font | Sarabun 14px / 26px regular |
| Icon size | 16×16px |
| Badge height | 18px |

---

## CSS Custom Properties

```css
:root {
  --sidenav-bg:             #fafafa;
  --sidenav-bg-hover:       #f1f0f0;
  --sidenav-border:         #e2e1e1;
  --sidenav-text:           #242424;
  --sidenav-text-subtle:    #625e5e;
  --sidenav-icon:           #625e5e;
  --sidenav-icon-active:    var(--btn-fill-bg, #2d79c8); /* color/primary/500☆ */
}
```

---

## CSS Classes

```css
/* ── Sidebar wrapper ──────────────────────────── */
.bos-sidenav {
  display: flex;
  flex-direction: column;
  background-color: var(--sidenav-bg, #fafafa);
  border-right: 1px solid var(--sidenav-border, #e2e1e1);
  width: 216px;
  height: 100%;
  position: relative;
  padding: 20px 8px;
  box-sizing: border-box;
}

.bos-sidenav--collapsed {
  width: 52px;
}

/* ── Section group ─────────────────────────────── */
.bos-sidenav__group {
  display: flex;
  flex-direction: column;
  gap: 4px;
  margin-bottom: 8px;
}

/* Section title (12px subtle) */
.bos-sidenav__title {
  display: flex;
  align-items: center;
  font-family: var(--font-family);
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  font-weight: var(--font-weight-regular);
  color: var(--sidenav-text-subtle, #625e5e);
  padding-left: 8px;
  height: 16px;
}

/* ── Nav item ──────────────────────────────────── */
.bos-sidenav__item {
  display: flex;
  align-items: center;
  gap: var(--space-xs, 4px);
  padding: var(--space-m, 8px);
  border-radius: var(--corner-xs, 4px);
  width: 200px;
  cursor: pointer;
  text-decoration: none;
  color: var(--sidenav-text, #242424);
  font-family: var(--font-family);
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height);
  font-weight: var(--font-weight-regular);
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.bos-sidenav__item:hover {
  background-color: var(--sidenav-bg-hover, #f1f0f0);
}

.bos-sidenav__item--active {
  background-color: var(--sidenav-bg, #fafafa);
}

.bos-sidenav__item--active .bos-sidenav__icon {
  color: var(--sidenav-icon-active, #2d79c8);
}

/* Icon */
.bos-sidenav__icon {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  color: var(--sidenav-icon, #625e5e);
}

/* Label */
.bos-sidenav__label {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Badge */
.bos-sidenav__badge {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 18px;
  padding: 0 8px;
  border-radius: 100px;
  background-color: #d2e4f8;
  color: #2d79c8;
  font-size: var(--text-small-size);  /* 12px */
  line-height: var(--text-small-line-height);
  font-weight: var(--font-weight-regular);
  flex-shrink: 0;
}

/* Right arrow */
.bos-sidenav__arrow {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  color: var(--sidenav-icon, #625e5e);
}

/* ── Bottom section (Settings) ─────────────────── */
.bos-sidenav__footer {
  position: absolute;
  bottom: 8px;
  left: 8px;
  display: flex;
  flex-direction: column;
  gap: var(--space-m, 8px);
  align-items: center;
  width: 200px;
}

.bos-sidenav__divider {
  height: 1px;
  width: 184px;
  background-color: var(--sidenav-border, #e2e1e1);
}

/* ── Sub-navigation popup ──────────────────────── */
.bos-sidenav__sub {
  position: absolute;
  left: 220px;
  top: 0;
  width: 123px;
  background-color: #fafafa;
  border: 1px solid #e2e1e1;
  border-radius: var(--corner-xs, 4px);
  padding: var(--space-m, 8px);
  display: flex;
  flex-direction: column;
  gap: var(--space-xs, 4px);
}

.bos-sidenav__sub-item {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--space-xs, 4px) var(--space-m, 8px);
  border-radius: var(--corner-xs, 4px);
  font-family: var(--font-family);
  font-size: var(--text-body-size);
  color: var(--sidenav-text, #242424);
  cursor: pointer;
}

.bos-sidenav__sub-item:hover {
  background-color: #f1f0f0;
}
```

---

## HTML Usage

```html
<!-- Full sidebar -->
<nav class="bos-sidenav">
  <!-- Section 1 -->
  <div class="bos-sidenav__group">
    <div class="bos-sidenav__title">
      <svg class="bos-sidenav__icon" width="16" height="16"><!-- Menu icon --></svg>
    </div>
    <span class="bos-sidenav__title">Title</span>

    <a href="#" class="bos-sidenav__item bos-sidenav__item--active">
      <svg class="bos-sidenav__icon" width="16" height="16"><!-- Icon --></svg>
      <span class="bos-sidenav__label">Navigate</span>
      <span class="bos-sidenav__badge">99</span>
      <svg class="bos-sidenav__arrow" width="16" height="16"><!-- Arrow right --></svg>
    </a>

    <a href="#" class="bos-sidenav__item">
      <svg class="bos-sidenav__icon" width="16" height="16"><!-- Icon --></svg>
      <span class="bos-sidenav__label">Navigate</span>
    </a>
  </div>

  <!-- Footer -->
  <div class="bos-sidenav__footer">
    <div class="bos-sidenav__divider"></div>
    <div>
      <a href="#" class="bos-sidenav__item">
        <svg class="bos-sidenav__icon" width="16" height="16"><!-- Icon --></svg>
        <span class="bos-sidenav__label">Setting</span>
      </a>
    </div>
  </div>
</nav>
```

---

## Usage Notes

- **Full mode** (216px): show label, badge, right arrow; `title` uses 12px subtle text.
- **Collapsed mode** (52px): show icon only; show tooltip on hover for label.
- **Active item** uses `color/primary/500☆` for the icon; text and background stay neutral.
- **Badge** uses secondary color (`#d2e4f8` / `#2d79c8`).
- **Sub-navigation** appears as an absolute popup to the right of the item on hover.
- Footer items (e.g., Settings, Logout) are separated from the main list by a 1px `#e2e1e1` divider.
