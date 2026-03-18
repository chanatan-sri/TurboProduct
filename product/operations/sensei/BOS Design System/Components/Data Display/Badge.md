# Badge

> **Page:** ↪︎ Badge
> **Type:** Component — Data Display

---

## Overview

BOS provides three badge variants: **Count** (numeric overlay), **Status** (dot + label), and **Dot** (colored indicator only).

---

## Variants

| Variant | Description |
|---|---|
| `Badge/Count` | Numeric count pill — appears near avatars or icons |
| `Badge/Status` | Colored dot + text label — shows current state |
| `Badge/Dot` | Small 6px dot only — compact status indicator |

---

## Badge / Count

### Sizes

| Size | Height | Padding |
|---|---|---|
| Small | 16px | 4px horizontal |
| Medium | 18px | 8px horizontal |

### Colors

| Color | Background | Text |
|---|---|---|
| `Secondary` | `#d2e4f8` | `#2d79c8` |

### CSS

```css
.bos-badge-count {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  height: 18px;
  padding: 0 8px;
  border-radius: 100px;
  background-color: #d2e4f8;
  color: #2d79c8;
  font-family: var(--font-family);
  font-size: var(--text-small-size);   /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  font-weight: var(--font-weight-regular);
  white-space: nowrap;
}

.bos-badge-count--sm { height: 16px; padding: 0 4px; }
```

### HTML

```html
<span class="bos-badge-count">99</span>
<span class="bos-badge-count bos-badge-count--sm">5</span>
```

---

## Badge / Status

A dot paired with a text label, used to indicate current status.

### Colors

| Type | Dot Color | Token |
|---|---|---|
| Default | `#e2e1e1` | `badge/dot/default` |
| Warning | `#eb6b00` | `badge/dot/warning` |
| Success | `#00a848` | `badge/dot/success` |
| Danger / Error | `#e62822` | `badge/dot/danger` |
| Info / Primary | `#2d79c8` | `badge/dot/primary` |

### CSS

```css
:root {
  --badge-dot-default: #e2e1e1;
  --badge-dot-warning: #eb6b00;
  --badge-dot-success: #00a848;
  --badge-dot-danger:  #e62822;
  --badge-dot-primary: #2d79c8;
  --badge-text:        #242424;   /* badge/text/visualization-primary */
}

.bos-badge-status {
  display: inline-flex;
  align-items: center;
  gap: var(--space-m, 8px);
  font-family: var(--font-family);
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height);
  font-weight: var(--font-weight-regular);
  color: var(--badge-text, #242424);
}

.bos-badge-status__dot {
  width: 6px;
  height: 6px;
  border-radius: 100px;
  flex-shrink: 0;
  background-color: var(--badge-dot-default, #e2e1e1);
}

/* Color modifiers */
.bos-badge-status--warning .bos-badge-status__dot { background-color: var(--badge-dot-warning, #eb6b00); }
.bos-badge-status--success .bos-badge-status__dot { background-color: var(--badge-dot-success, #00a848); }
.bos-badge-status--danger  .bos-badge-status__dot { background-color: var(--badge-dot-danger, #e62822); }
.bos-badge-status--primary .bos-badge-status__dot { background-color: var(--badge-dot-primary, #2d79c8); }
```

### HTML

```html
<div class="bos-badge-status bos-badge-status--warning">
  <span class="bos-badge-status__dot"></span>
  Warning
</div>

<div class="bos-badge-status bos-badge-status--success">
  <span class="bos-badge-status__dot"></span>
  Active
</div>

<div class="bos-badge-status bos-badge-status--danger">
  <span class="bos-badge-status__dot"></span>
  Error
</div>
```

---

## Badge / Dot

A standalone 6px colored dot — no text.

```css
.bos-badge-dot {
  width: 6px;
  height: 6px;
  border-radius: 100px;
  background-color: var(--badge-dot-default, #e2e1e1);
  display: inline-block;
  flex-shrink: 0;
}

.bos-badge-dot--warning { background-color: var(--badge-dot-warning, #eb6b00); }
.bos-badge-dot--success { background-color: var(--badge-dot-success, #00a848); }
.bos-badge-dot--danger  { background-color: var(--badge-dot-danger, #e62822); }
.bos-badge-dot--primary { background-color: var(--badge-dot-primary, #2d79c8); }
```

---

## Usage Notes

- **Count** badge appears adjacent to notifications or avatar icons.
- **Status** badge is used in list items and cards to communicate current state.
- **Dot** badge alone is used for compact status where label space is unavailable.
- Use Auto layout padding to adjust badge offset when positioned over other elements.
