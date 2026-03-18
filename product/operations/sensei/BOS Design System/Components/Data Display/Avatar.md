# Avatar

> **Page:** ↪︎ Avatar
> **Type:** Component — Data Display

---

## Overview

Avatars represent people or objects. Supports images, icons, or letter placeholders. Comes in 2 shapes × 4 sizes, with optional badge overlays.

---

## Shapes

| Shape | Corner Radius |
|---|---|
| `circle` | `var(--corner/full, 9999px)` |
| `square` | `var(--corner/xs, 4px)` |

---

## Sizes

| Size | Dimensions |
|---|---|
| Small | 24 × 24px |
| Medium | 32 × 32px |
| Large | 40 × 40px |
| Custom | 64 × 64px |

---

## Types

| Type | Description |
|---|---|
| `image` | Photo/image fills the avatar area |
| `icon` | Icon centered on tinted background |

---

## Badge Variants

| Badge | Description |
|---|---|
| `none` | No badge |
| `numeric` | Count badge (top-right) |
| `dot` | Small colored dot indicator (top-right) |

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `background-image` | `color/primary/50` | `#e8f2fb` | Icon-type avatar background |
| `background-placeholder` | `color/neutral/200` | `#e2e1e1` | Placeholder fallback bg |
| `icon-default` | `color/neutral/0-white` | `#ffffff` | Icon color on tinted bg |

---

## CSS Custom Properties

```css
:root {
  --avatar-bg-image:       #e8f2fb;   /* color/primary/50 */
  --avatar-bg-placeholder: #e2e1e1;   /* color/neutral/200 */
  --avatar-icon:           #ffffff;   /* color/neutral/0-white */
}
```

---

## CSS Classes

```css
/* ── Base ──────────────────────────────────────── */
.bos-avatar {
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  flex-shrink: 0;
  position: relative;
}

/* ── Shapes ─────────────────────────────────────── */
.bos-avatar--circle { border-radius: var(--corner-full, 9999px); }
.bos-avatar--square { border-radius: var(--corner-xs, 4px); }

/* ── Sizes ──────────────────────────────────────── */
.bos-avatar--sm { width: 24px;  height: 24px; }
.bos-avatar--md { width: 32px;  height: 32px; }
.bos-avatar--lg { width: 40px;  height: 40px; }
.bos-avatar--custom { width: 64px; height: 64px; }

/* ── Image type ─────────────────────────────────── */
.bos-avatar--image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* ── Icon type ──────────────────────────────────── */
.bos-avatar--icon {
  background-color: var(--avatar-bg-image, #e8f2fb);
  color: var(--avatar-icon, #ffffff);
}

.bos-avatar--icon svg {
  width: 60%;
  height: 60%;
}

/* ── Badge wrapper ──────────────────────────────── */
.bos-avatar-wrap {
  position: relative;
  display: inline-flex;
}

/* Numeric badge */
.bos-avatar-wrap__badge-count {
  position: absolute;
  top: -4px;
  right: -4px;
  min-width: 16px;
  height: 16px;
  padding: 0 4px;
  border-radius: 100px;
  background-color: #d2e4f8;
  color: #2d79c8;
  font-family: var(--font-family);
  font-size: 10px;
  line-height: 16px;
  text-align: center;
  white-space: nowrap;
  border: 2px solid #ffffff;
}

/* Dot badge */
.bos-avatar-wrap__badge-dot {
  position: absolute;
  top: 0;
  right: 0;
  width: 8px;
  height: 8px;
  border-radius: 100px;
  background-color: #e62822;   /* default red dot */
  border: 2px solid #ffffff;
}
```

---

## HTML Usage

```html
<!-- Circle image — medium -->
<div class="bos-avatar bos-avatar--circle bos-avatar--md bos-avatar--image">
  <img src="photo.jpg" alt="User Name" />
</div>

<!-- Circle icon — medium -->
<div class="bos-avatar bos-avatar--circle bos-avatar--md bos-avatar--icon">
  <svg width="16" height="16"><!-- icon --></svg>
</div>

<!-- Square image — large -->
<div class="bos-avatar bos-avatar--square bos-avatar--lg bos-avatar--image">
  <img src="photo.jpg" alt="User Name" />
</div>

<!-- With numeric badge -->
<div class="bos-avatar-wrap">
  <div class="bos-avatar bos-avatar--circle bos-avatar--md bos-avatar--image">
    <img src="photo.jpg" alt="User" />
  </div>
  <span class="bos-avatar-wrap__badge-count">3</span>
</div>

<!-- With dot badge -->
<div class="bos-avatar-wrap">
  <div class="bos-avatar bos-avatar--circle bos-avatar--md bos-avatar--image">
    <img src="photo.jpg" alt="User" />
  </div>
  <span class="bos-avatar-wrap__badge-dot"></span>
</div>
```

---

## Usage Notes

- Always provide `alt` text on the `<img>` for accessibility.
- For icon avatars, ensure icon has sufficient contrast on the tinted background.
- Badge is positioned top-right; use a white border to separate it from the avatar.
- `custom` size (64px) is used in profile headers and larger layout contexts.
