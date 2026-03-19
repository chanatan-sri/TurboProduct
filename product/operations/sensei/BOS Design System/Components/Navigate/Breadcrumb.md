# Breadcrumb

> **Page:** ↪︎ Breadcrumb
> **Type:** Component — Navigate

---

## Overview

Breadcrumb shows the user's current location within a hierarchy. Items are separated by a `/` character. The last item is the active (current) page.

---

## States

| State | Description |
|---|---|
| `Default` | Inactive link — muted text |
| `Hover` | Hovered link — dark text, neutral background |
| `Active` | Current page — dark bold text |

---

## Design Tokens

### Breadcrumb / Background

| Token | BOS color | Usage |
|---|---|---|
| `background-hover` | `color/neutral/50` | Background when hovered |

### Breadcrumb / Text

| Token | BOS color | Hex | Usage |
|---|---|---|---|
| `text-default` | `color/neutral/600` | `#625e5e` | Inactive breadcrumb link |
| `text-hover` | `color/neutral/900-black` | `#242424` | Hovered breadcrumb link |
| `text-active` | `color/neutral/900-black` | `#242424` | Current / active page |

---

## CSS Custom Properties

```css
:root {
  --breadcrumb-text-default:     #625e5e;  /* color/neutral/600 */
  --breadcrumb-text-hover:       #242424;  /* color/neutral/900-black */
  --breadcrumb-text-active:      #242424;  /* color/neutral/900-black */
  --breadcrumb-bg-hover:         #fafafa;  /* color/neutral/50 */
}
```

---

## CSS Classes

```css
/* Breadcrumb container */
.bos-breadcrumb {
  display: flex;
  align-items: center;
  gap: var(--space-xs, 4px);          /* 4px between items + separators */
  font-family: var(--font-family);
  font-size: var(--text-body-size);   /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
}

/* Individual breadcrumb item */
.bos-breadcrumb__item {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--space-2xs, 2px);     /* 2px padding around item */
  color: var(--breadcrumb-text-default, #625e5e);
  text-decoration: none;
  border-radius: var(--corner-xs, 4px);
  white-space: nowrap;
  cursor: pointer;
}

.bos-breadcrumb__item:hover {
  color: var(--breadcrumb-text-hover, #242424);
  background-color: var(--breadcrumb-bg-hover, #fafafa);
}

.bos-breadcrumb__item--active {
  color: var(--breadcrumb-text-active, #242424);
  font-weight: var(--font-weight-semibold);
  cursor: default;
  pointer-events: none;
}

/* Separator "/" */
.bos-breadcrumb__separator {
  color: var(--breadcrumb-text-default, #625e5e);
  white-space: nowrap;
  user-select: none;
}
```

---

## HTML Usage

```html
<nav class="bos-breadcrumb" aria-label="breadcrumb">
  <a href="#" class="bos-breadcrumb__item">Home</a>
  <span class="bos-breadcrumb__separator">/</span>
  <a href="#" class="bos-breadcrumb__item">Category</a>
  <span class="bos-breadcrumb__separator">/</span>
  <span class="bos-breadcrumb__item bos-breadcrumb__item--active" aria-current="page">Current Page</span>
</nav>
```

---

## Usage Notes

- The last item should always be `--active` and have `aria-current="page"`.
- Use `bos-breadcrumb__separator` for the `/` separator — do not put the slash inside the item.
- Breadcrumb items are body text (14px / 26px) regular weight; the active item uses semibold.
