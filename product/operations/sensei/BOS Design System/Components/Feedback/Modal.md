# Modal

> **Page:** ↪︎ Modal
> **Type:** Component — Feedback

---

## Overview

A dialog overlay for focused tasks and confirmations. Consists of a header (title + close icon), a body (swappable content slot — any component can be placed here, e.g. radio box group, form fields), and a footer (action buttons). Three sizes. An **Inform modal** variant has an icon + title in the header instead of a plain title.

---

## Sizes

| Size | Width |
|---|---|
| `Small` | 400px |
| `Medium` | 600px |
| `Large` | 800px |

---

## Structure

| Section | Description |
|---|---|
| Header | Title (semibold) + close icon (16px), top corners rounded |
| Body | Content slot — swap in any component |
| Footer | Right-aligned buttons: subtle (outline) + main (primary fill) |

---

## Dimensions

| Property | Value |
|---|---|
| Corner radius | 8px (all four corners) |
| Header/Footer padding | 12px |
| Body padding | 12px |
| Close icon | 16 × 16px |
| Header title font | 14px/26px semibold (Basic modal) |
| Header title font | 16px/28px regular (Inform modal) |
| Body text font | 14px/26px regular |

---

## Colors

| Element | Color |
|---|---|
| Background | `#fafafa` |
| Border | `#e2e1e1` |
| Title text | `#242424` |
| Body text | `#242424` |
| Overlay backdrop | `rgba(0, 0, 0, 0.45)` |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `modals/background/default` | `#fafafa` | Modal bg |
| `modals/stroke/default` | `#e2e1e1` | Modal border |
| `modals/text/default` | `#242424` | All text |

---

## CSS Custom Properties

```css
:root {
  --modal-bg:       #fafafa;
  --modal-border:   #e2e1e1;
  --modal-text:     #242424;
  --modal-overlay:  rgba(0, 0, 0, 0.45);
}
```

---

## CSS Classes

```css
/* ── Overlay backdrop ─────────────────────── */
.bos-modal-overlay {
  position: fixed;
  inset: 0;
  background-color: var(--modal-overlay, rgba(0, 0, 0, 0.45));
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  font-family: var(--font-family);
}

/* ── Modal container ──────────────────────── */
.bos-modal {
  display: flex;
  flex-direction: column;
  background-color: var(--modal-bg, #fafafa);
  border: 1px solid var(--modal-border, #e2e1e1);
  border-radius: var(--corner-m, 8px);
  width: 600px;   /* default Medium */
}

.bos-modal--sm { width: 400px; }
.bos-modal--md { width: 600px; }
.bos-modal--lg { width: 800px; }

/* ── Header ───────────────────────────────── */
.bos-modal__header {
  display: flex;
  align-items: center;
  gap: var(--space-m, 8px);
  padding: var(--space-l, 12px);
  border-bottom: 1px solid var(--modal-border, #e2e1e1);
}

.bos-modal__title {
  flex: 1;
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-semibold, 600);
  color: var(--modal-text, #242424);
  min-width: 0;
}

.bos-modal__close {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  cursor: pointer;
  color: var(--modal-text, #242424);
}

/* ── Body ─────────────────────────────────── */
.bos-modal__body {
  flex: 1;
  padding: var(--space-l, 12px);
  font-size: var(--text-body-size);
  line-height: var(--text-body-line-height);
  color: var(--modal-text, #242424);
}

/* ── Footer ───────────────────────────────── */
.bos-modal__footer {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: var(--space-m, 8px);
  padding: var(--space-m, 8px) var(--space-l, 12px) var(--space-l, 12px);
  border-top: 1px solid var(--modal-border, #e2e1e1);
}

/* ── Inform modal header (icon + title) ───── */
.bos-modal__header--inform {
  flex-direction: column;
  align-items: flex-start;
  gap: var(--space-m, 8px);
}

.bos-modal__header-top {
  display: flex;
  align-items: center;
  gap: var(--space-m, 8px);
  width: 100%;
}

.bos-modal__header-icon {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.bos-modal__title--inform {
  font-size: 16px;
  line-height: 28px;
  font-weight: var(--font-weight-regular);
  color: var(--modal-text, #242424);
}

.bos-modal__header-content {
  padding-left: 28px; /* icon (20px) + gap (8px) */
  font-size: var(--text-body-size);
  line-height: var(--text-body-line-height);
  color: var(--modal-text, #242424);
  width: 100%;
}
```

---

## HTML Usage

```html
<!-- Basic modal — Medium size -->
<div class="bos-modal-overlay">
  <div class="bos-modal bos-modal--md">
    <!-- Header -->
    <div class="bos-modal__header">
      <span class="bos-modal__title">Title</span>
      <svg class="bos-modal__close" width="16" height="16"><!-- × icon --></svg>
    </div>
    <!-- Body — swap any component here -->
    <div class="bos-modal__body">
      Content
    </div>
    <!-- Footer -->
    <div class="bos-modal__footer">
      <button class="bos-btn bos-btn--outline-secondary">subtle</button>
      <button class="bos-btn bos-btn--primary">main</button>
    </div>
  </div>
</div>

<!-- Basic modal — Small size -->
<div class="bos-modal-overlay">
  <div class="bos-modal bos-modal--sm">
    <div class="bos-modal__header">
      <span class="bos-modal__title">Title</span>
      <svg class="bos-modal__close" width="16" height="16"><!-- × icon --></svg>
    </div>
    <div class="bos-modal__body">Content</div>
    <div class="bos-modal__footer">
      <button class="bos-btn bos-btn--outline-secondary">subtle</button>
      <button class="bos-btn bos-btn--primary">main</button>
    </div>
  </div>
</div>

<!-- Inform modal (icon in header) -->
<div class="bos-modal-overlay">
  <div class="bos-modal bos-modal--md">
    <div class="bos-modal__header">
      <svg class="bos-modal__close" width="16" height="16" style="margin-left:auto;"><!-- × --></svg>
      <div style="width:100%; display:flex; flex-direction:column; gap:8px;">
        <div style="display:flex; align-items:center; gap:8px;">
          <svg class="bos-modal__header-icon" width="20" height="20"><!-- error icon --></svg>
          <span class="bos-modal__title--inform">Title</span>
        </div>
        <div class="bos-modal__header-content">Content</div>
      </div>
    </div>
    <div class="bos-modal__footer">
      <button class="bos-btn bos-btn--outline-secondary">subtle</button>
      <button class="bos-btn bos-btn--primary">main</button>
    </div>
  </div>
</div>
```

---

## Usage Notes

- The **body content slot** (`bos-modal__body`) accepts any component — radio box group, form fields, tables, etc. This is the "Swap" area shown in Figma.
- Buttons in the footer are always right-aligned: subtle (outline secondary) on the left, main (primary fill) on the right.
- The **Inform modal** variant replaces the plain title row with a semantic icon + larger title (16px) and indented body content (padded left by 28px to align under the icon).
- Always use an overlay backdrop (`bos-modal-overlay`) to block page interaction.
- The close icon is 16px; clicking it dismisses the modal.
