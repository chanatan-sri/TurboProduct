# Alert Bar

> **Page:** ↪︎ Alert bar
> **Type:** Component — Feedback

---

## Overview

A full-width banner for contextual feedback messages. Supports four semantic types (Information, Success, Warning, Error), an optional description line, an optional left icon, and an optional action button on the right.

---

## Types

| Type | Background | Border | Token |
|---|---|---|---|
| `Information` | `#f2f7fc` | `#d2e4f8` | `color/primary/50`, `color/primary/100` |
| `Success` | `#e7fbec` | `#b1f0c2` | `color/success/50`, `color/success/100` |
| `Warning` | `#fffbe6` | `#ffe7a1` | `color/warning/50`, `color/warning/100` |
| `Error` | `#fff2f0` | `#ffccc7` | `color/error/50`, `color/error/100` |

---

## Variants

| Variant | Height | Description |
|---|---|---|
| `With description` | 58px | Title + description line |
| `No description` | 48px | Title only |

---

## Dimensions

| Property | Value |
|---|---|
| Width | 100% (fluid) |
| Corner radius | 6px |
| Padding | 16px horizontal, 8px vertical |
| Left icon | 20 × 20px |
| Gap (icon → content) | 4px |
| Shadow | `0px 4px 6px 0px rgba(0,0,0,0.05)` |

---

## Typography

| Element | Size | Weight | Color |
|---|---|---|---|
| Title | 14px/26px | Semibold (600) | `#242424` |
| Description | 12px/22px | Regular | `#625e5e` |
| Action button | 14px/26px | Regular | `#242424` |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `alert/background/info` | `#f2f7fc` | Info bg |
| `alert/border/info` | `#d2e4f8` | Info border |
| `alert/background/success` | `#e7fbec` | Success bg |
| `alert/border/success` | `#b1f0c2` | Success border |
| `alert/background/warning` | `#fffbe6` | Warning bg |
| `alert/border/warning` | `#ffe7a1` | Warning border |
| `alert/background/error` | `#fff2f0` | Error bg |
| `alert/border/error` | `#ffccc7` | Error border |
| `color/neutral/900-black` | `#242424` | Title text |
| `color/neutral/600` | `#625e5e` | Description text |

---

## CSS Custom Properties

```css
:root {
  --alert-bg-info:      #f2f7fc;
  --alert-border-info:  #d2e4f8;
  --alert-bg-success:   #e7fbec;
  --alert-border-success: #b1f0c2;
  --alert-bg-warning:   #fffbe6;
  --alert-border-warning: #ffe7a1;
  --alert-bg-error:     #fff2f0;
  --alert-border-error: #ffccc7;
  --alert-title:        #242424;
  --alert-description:  #625e5e;
}
```

---

## CSS Classes

```css
/* ── Base alert bar ───────────────────────── */
.bos-alert {
  display: flex;
  align-items: flex-start;
  gap: var(--space-xs, 4px);
  width: 100%;
  padding: var(--space-m, 8px) var(--space-xl, 16px);
  border: 1px solid transparent;
  border-radius: var(--corner-s, 6px);
  box-shadow: 0px 4px 6px 0px rgba(0, 0, 0, 0.05);
  font-family: var(--font-family);
}

/* ── Types ────────────────────────────────── */
.bos-alert--info {
  background-color: var(--alert-bg-info, #f2f7fc);
  border-color: var(--alert-border-info, #d2e4f8);
}

.bos-alert--success {
  background-color: var(--alert-bg-success, #e7fbec);
  border-color: var(--alert-border-success, #b1f0c2);
}

.bos-alert--warning {
  background-color: var(--alert-bg-warning, #fffbe6);
  border-color: var(--alert-border-warning, #ffe7a1);
}

.bos-alert--error {
  background-color: var(--alert-bg-error, #fff2f0);
  border-color: var(--alert-border-error, #ffccc7);
}

/* ── Left icon ────────────────────────────── */
.bos-alert__icon {
  display: flex;
  align-items: center;
  padding: var(--space-2xs, 2px) 0;
  flex-shrink: 0;
}

.bos-alert__icon svg,
.bos-alert__icon img {
  width: 20px;
  height: 20px;
}

/* ── Content ──────────────────────────────── */
.bos-alert__content {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 24px;
  min-width: 0;
}

/* ── Title ────────────────────────────────── */
.bos-alert__title {
  display: flex;
  align-items: center;
  height: 24px;
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-semibold, 600);
  color: var(--alert-title, #242424);
}

/* ── Description ──────────────────────────── */
.bos-alert__description {
  display: flex;
  align-items: center;
  padding-bottom: var(--space-2xs, 2px);
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  font-weight: var(--font-weight-regular);
  color: var(--alert-description, #625e5e);
}

/* ── Action button ────────────────────────── */
.bos-alert__action {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 32px;
  min-width: 80px;
  padding: 0 var(--space-l, 12px);
  background-color: #fafafa;
  border: 1px solid #e2e1e1;
  border-radius: var(--corner-m, 8px);
  font-size: var(--text-body-size);
  font-weight: var(--font-weight-regular);
  color: #242424;
  cursor: pointer;
  white-space: nowrap;
  flex-shrink: 0;
}
```

---

## HTML Usage

```html
<!-- Information — with description + action -->
<div class="bos-alert bos-alert--info">
  <div class="bos-alert__icon">
    <svg width="20" height="20"><!-- info icon --></svg>
  </div>
  <div class="bos-alert__content">
    <div class="bos-alert__title">Title</div>
    <div class="bos-alert__description">Description</div>
  </div>
  <button class="bos-alert__action">Button</button>
</div>

<!-- Success — with description -->
<div class="bos-alert bos-alert--success">
  <div class="bos-alert__icon">
    <svg width="20" height="20"><!-- check icon --></svg>
  </div>
  <div class="bos-alert__content">
    <div class="bos-alert__title">สำเร็จ</div>
    <div class="bos-alert__description">บันทึกข้อมูลเรียบร้อยแล้ว</div>
  </div>
</div>

<!-- Warning — no description -->
<div class="bos-alert bos-alert--warning">
  <div class="bos-alert__icon">
    <svg width="20" height="20"><!-- warning icon --></svg>
  </div>
  <div class="bos-alert__content">
    <div class="bos-alert__title">โปรดตรวจสอบข้อมูลก่อนบันทึก</div>
  </div>
</div>

<!-- Error — with description + action -->
<div class="bos-alert bos-alert--error">
  <div class="bos-alert__icon">
    <svg width="20" height="20"><!-- error icon --></svg>
  </div>
  <div class="bos-alert__content">
    <div class="bos-alert__title">เกิดข้อผิดพลาด</div>
    <div class="bos-alert__description">ไม่สามารถบันทึกข้อมูลได้ กรุณาลองใหม่อีกครั้ง</div>
  </div>
  <button class="bos-alert__action">ลองใหม่</button>
</div>
```

---

## Usage Notes

- Alert bar spans full width of its container — do not set a fixed width.
- The left icon is always 20px; use the semantic icon for each type (info / check / warning / error circle).
- The action button is optional — include only when a direct action is available.
- Use **with description** variant when additional context is needed; omit `bos-alert__description` for title-only.
- Corner radius is always 6px — do not change.
