# Header

> **Page:** ↪︎ Header
> **Type:** Component — Data Display

---

## Overview

The top navigation header bar. Contains a page title on the left, and a user identity section (avatar, name, branch) plus action buttons on the right. Also supports sub-header variants for contextual actions.

---

## Variants

| Variant | Description |
|---|---|
| `Property 1=1` | Standard header — title + user info + logout |
| `Property 1=2` | Header with notification bell |
| `Property 1=Variant3` | Extended variant |
| Sub-header | Secondary bar below main header — button sets for page-level actions |

---

## Dimensions

| Property | Value |
|---|---|
| Height | 48px |
| Width | Full width (1920px / 1868px content) |
| Horizontal padding | `var(--space/2xl, 20px)` |
| Gap between sections | `var(--space/xl, 16px)` |

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `background-default` | `color/neutral/0-white` | `#ffffff` | Main header bg |
| `icon-default` | `color/neutral/600` | `#625e5e` | Action icon color |
| `text-default` | `color/neutral/900-black` | `#242424` | Title and user name text |
| `divider-default` | `color/neutral/100` | `#e2e1e1` | Bottom border + vertical dividers |

---

## CSS Custom Properties

```css
:root {
  --header-bg:      #ffffff;   /* color/neutral/0-white */
  --header-border:  #e2e1e1;   /* color/neutral/100 */
  --header-text:    #242424;   /* color/neutral/900-black */
  --header-icon:    #625e5e;   /* color/neutral/600 */
}
```

---

## CSS Classes

```css
/* ── Header bar ─────────────────────────────────── */
.bos-header {
  display: flex;
  align-items: center;
  gap: var(--space-xl, 16px);
  padding: 0 var(--space-2xl, 20px);
  height: 48px;
  width: 100%;
  background-color: var(--header-bg, #ffffff);
  border-bottom: 1px solid var(--header-border, #e2e1e1);
  box-sizing: border-box;
}

/* ── Title ──────────────────────────────────────── */
.bos-header__title {
  flex: 1;
  font-family: var(--font-family);
  font-size: 16px;              /* scale/400 */
  line-height: 28px;            /* scale/700 */
  font-weight: var(--font-weight-semibold, 600);
  color: var(--header-text, #242424);
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* ── Right section ──────────────────────────────── */
.bos-header__actions {
  display: flex;
  align-items: center;
  gap: 16px;
  flex-shrink: 0;
}

/* ── User info ──────────────────────────────────── */
.bos-header__user {
  display: flex;
  align-items: center;
  gap: var(--space-xs, 4px);
}

.bos-header__user-name,
.bos-header__user-branch {
  font-family: var(--font-family);
  font-size: var(--text-body-size);    /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--header-text, #242424);
  white-space: nowrap;
}

/* ── Vertical divider ───────────────────────────── */
.bos-header__divider {
  width: 1px;
  height: 24px;
  background-color: var(--header-border, #e2e1e1);
  flex-shrink: 0;
}

/* ── Icon button ────────────────────────────────── */
.bos-header__icon-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  border-radius: var(--corner-xs, 4px);
  border: none;
  cursor: pointer;
  flex-shrink: 0;
}

.bos-header__icon-btn--fill {
  background-color: #2d79c8;
  color: #ffffff;
}

.bos-header__icon-btn--outline {
  background-color: #fafafa;
  border: 1px solid #e2e1e1;
  color: #625e5e;
}

/* ── Sub-header ─────────────────────────────────── */
.bos-sub-header {
  display: flex;
  align-items: center;
  gap: var(--space-m, 8px);
  padding: 0 var(--space-2xl, 20px);
  height: 60px;
  width: 100%;
  background-color: #ffffff;
  border-bottom: 1px solid #e2e1e1;
  box-sizing: border-box;
}
```

---

## HTML Usage

```html
<!-- Standard Header -->
<header class="bos-header">
  <span class="bos-header__title">Page Title</span>

  <div class="bos-header__actions">
    <!-- Notification button -->
    <button class="bos-header__icon-btn bos-header__icon-btn--fill" aria-label="Notifications">
      <svg width="16" height="16"><!-- bell icon --></svg>
    </button>

    <!-- User section -->
    <div class="bos-header__user">
      <!-- Avatar -->
      <div class="bos-avatar bos-avatar--circle bos-avatar--sm bos-avatar--image">
        <img src="avatar.jpg" alt="User" />
      </div>
      <span class="bos-header__user-name">เงินตรา มากมี</span>
    </div>

    <div class="bos-header__divider"></div>

    <span class="bos-header__user-branch">เคหะร่มเกล้า</span>

    <!-- Logout button -->
    <button class="bos-header__icon-btn bos-header__icon-btn--outline" aria-label="Logout">
      <svg width="16" height="16"><!-- logout icon --></svg>
    </button>
  </div>
</header>

<!-- Sub-header -->
<div class="bos-sub-header">
  <button class="bos-btn bos-btn--fill">Action</button>
  <button class="bos-btn bos-btn--outline-secondary">Cancel</button>
</div>
```

---

## Usage Notes

- The header spans full viewport width.
- Title truncates with ellipsis on overflow.
- User name and branch name are separated by the right section layout — not by a divider in all variants.
- Sub-header appears directly below the main header and carries page-level actions (submit, cancel, etc.).
