# Toast

> **Page:** ↪︎ Toast
> **Type:** Component — Feedback

---

## Overview

A small non-blocking notification that appears temporarily (usually bottom-right or top-right). Supports four semantic states and two styles: **Fill** (solid colored background) and **Outline** (light background with colored border). Contains a prefix icon, header (semibold), optional content text, and a close icon.

---

## Styles

| Style | Description |
|---|---|
| `Fill` | Solid colored background — high emphasis |
| `Outline` | Light background + colored border — low emphasis |

---

## States

| State | Fill bg | Outline bg | Border |
|---|---|---|---|
| `Info` | `#2d79c8` | `#f2f7fc` | `#2d79c8` |
| `Success` | `color/success/500☆` | `color/success/50` | `color/success/500☆` |
| `Warning` | `color/warning/400` | `color/warning/50` | `color/warning/400` |
| `Error` | `#e62822` | `#fff2f0` | `#e62822` |

---

## Text Colors

| Style | Color |
|---|---|
| `Fill` — all text & icons | `#fafafa` (white) |
| `Outline` — all text & icons | `#242424` (neutral/900-black) |

---

## Dimensions

| Property | Value |
|---|---|
| Width | 376px |
| Min height | 66px (with content), 42px (header only) |
| Corner radius | 8px |
| Padding | 8px |
| Gap (icon → content) | 6px |
| Prefix icon | 24 × 24px |
| Close icon | 24 × 24px |
| Shadow | `0px 4px 6px 0px rgba(0,0,0,0.05)` |

---

## Typography

| Element | Size | Weight |
|---|---|---|
| Header | 14px/26px | Semibold (600) |
| Content | 14px/26px | Regular |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `toast/background/info` | `#2d79c8` | Info fill bg |
| `toast/background/info-alternative` | `#f2f7fc` | Info outline bg |
| `toast/stroke/info` | `#2d79c8` | Info outline border |
| `toast/background/error` | `#e62822` | Error fill bg |
| `toast/text/success` | `#fafafa` | Fill text (all states) |
| `toast/text/alternative` | `#242424` | Outline text (all states) |
| `color/neutral/0-white` | `#fafafa` | Fill icons |

---

## CSS Custom Properties

```css
:root {
  /* Info */
  --toast-bg-info:              #2d79c8;
  --toast-bg-info-outline:      #f2f7fc;
  --toast-border-info:          #2d79c8;
  /* Success */
  --toast-bg-success:           #008A2B;
  --toast-bg-success-outline:   #e7fbec;
  --toast-border-success:       #008A2B;
  /* Warning */
  --toast-bg-warning:           #EB6B00;
  --toast-bg-warning-outline:   #fffbe6;
  --toast-border-warning:       #EB6B00;
  /* Error */
  --toast-bg-error:             #e62822;
  --toast-bg-error-outline:     #fff2f0;
  --toast-border-error:         #e62822;
  /* Text */
  --toast-text-fill:            #fafafa;
  --toast-text-outline:         #242424;
}
```

---

## CSS Classes

```css
/* ── Base toast ───────────────────────────── */
.bos-toast {
  display: flex;
  align-items: flex-start;
  gap: var(--space-s, 6px);
  width: 376px;
  padding: var(--space-m, 8px);
  border-radius: var(--corner-m, 8px);
  box-shadow: 0px 4px 6px 0px rgba(0, 0, 0, 0.05);
  font-family: var(--font-family);
  border: 1px solid transparent;
}

/* ── Prefix icon ──────────────────────────── */
.bos-toast__icon {
  width: 24px;
  height: 24px;
  flex-shrink: 0;
}

/* ── Content area ─────────────────────────── */
.bos-toast__content {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 24px;
  min-width: 0;
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
}

.bos-toast__header {
  display: flex;
  align-items: center;
  min-height: 24px;
  font-weight: var(--font-weight-semibold, 600);
}

.bos-toast__body {
  font-weight: var(--font-weight-regular);
}

/* ── Close icon ───────────────────────────── */
.bos-toast__close {
  display: flex;
  align-items: center;
  min-height: 24px;
  flex-shrink: 0;
  width: 24px;
  height: 24px;
  cursor: pointer;
}

/* ── Fill style ───────────────────────────── */
.bos-toast--fill { color: var(--toast-text-fill, #fafafa); }

.bos-toast--fill.bos-toast--info    { background-color: var(--toast-bg-info, #2d79c8); }
.bos-toast--fill.bos-toast--success { background-color: var(--toast-bg-success, #008A2B); }
.bos-toast--fill.bos-toast--warning { background-color: var(--toast-bg-warning, #EB6B00); }
.bos-toast--fill.bos-toast--error   { background-color: var(--toast-bg-error, #e62822); }

/* ── Outline style ────────────────────────── */
.bos-toast--outline { color: var(--toast-text-outline, #242424); }

.bos-toast--outline.bos-toast--info {
  background-color: var(--toast-bg-info-outline, #f2f7fc);
  border-color: var(--toast-border-info, #2d79c8);
}
.bos-toast--outline.bos-toast--success {
  background-color: var(--toast-bg-success-outline, #e7fbec);
  border-color: var(--toast-border-success, #008A2B);
}
.bos-toast--outline.bos-toast--warning {
  background-color: var(--toast-bg-warning-outline, #fffbe6);
  border-color: var(--toast-border-warning, #EB6B00);
}
.bos-toast--outline.bos-toast--error {
  background-color: var(--toast-bg-error-outline, #fff2f0);
  border-color: var(--toast-border-error, #e62822);
}

/* ── Toast list (stacked toasts) ──────────── */
.bos-toast-list {
  position: fixed;
  bottom: 24px;
  right: 24px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  z-index: 1100;
}
```

---

## HTML Usage

```html
<!-- Info — Fill -->
<div class="bos-toast bos-toast--fill bos-toast--info">
  <svg class="bos-toast__icon" width="24" height="24"><!-- info icon --></svg>
  <div class="bos-toast__content">
    <div class="bos-toast__header">header</div>
    <div class="bos-toast__body">Content</div>
  </div>
  <div class="bos-toast__close">
    <svg width="24" height="24"><!-- × icon --></svg>
  </div>
</div>

<!-- Success — Fill -->
<div class="bos-toast bos-toast--fill bos-toast--success">
  <svg class="bos-toast__icon" width="24" height="24"><!-- check icon --></svg>
  <div class="bos-toast__content">
    <div class="bos-toast__header">บันทึกสำเร็จ</div>
    <div class="bos-toast__body">ข้อมูลถูกบันทึกเรียบร้อยแล้ว</div>
  </div>
  <div class="bos-toast__close">
    <svg width="24" height="24"><!-- × icon --></svg>
  </div>
</div>

<!-- Warning — Fill -->
<div class="bos-toast bos-toast--fill bos-toast--warning">
  <svg class="bos-toast__icon" width="24" height="24"><!-- warning icon --></svg>
  <div class="bos-toast__content">
    <div class="bos-toast__header">คำเตือน</div>
  </div>
  <div class="bos-toast__close">
    <svg width="24" height="24"><!-- × icon --></svg>
  </div>
</div>

<!-- Error — Fill -->
<div class="bos-toast bos-toast--fill bos-toast--error">
  <svg class="bos-toast__icon" width="24" height="24"><!-- error icon --></svg>
  <div class="bos-toast__content">
    <div class="bos-toast__header">เกิดข้อผิดพลาด</div>
    <div class="bos-toast__body">ไม่สามารถดำเนินการได้</div>
  </div>
  <div class="bos-toast__close">
    <svg width="24" height="24"><!-- × icon --></svg>
  </div>
</div>

<!-- Info — Outline -->
<div class="bos-toast bos-toast--outline bos-toast--info">
  <svg class="bos-toast__icon" width="24" height="24"><!-- info icon --></svg>
  <div class="bos-toast__content">
    <div class="bos-toast__header">header</div>
    <div class="bos-toast__body">Content</div>
  </div>
  <div class="bos-toast__close">
    <svg width="24" height="24"><!-- × icon --></svg>
  </div>
</div>

<!-- Toast list container -->
<div class="bos-toast-list">
  <div class="bos-toast bos-toast--fill bos-toast--success"><!-- ... --></div>
  <div class="bos-toast bos-toast--fill bos-toast--error"><!-- ... --></div>
</div>
```

---

## Usage Notes

- Toasts are fixed-position, typically bottom-right (`bos-toast-list`).
- Width is always 376px — do not stretch to full width.
- Use **Fill** for high-emphasis feedback (errors, critical success). Use **Outline** for low-emphasis or informational messages.
- The content (sub-header) line is optional — omit `bos-toast__body` for header-only toasts.
- Auto-dismiss after 3–5 seconds is recommended; always include a close icon for manual dismissal.
- Stack multiple toasts using `bos-toast-list` with 8px gap.
