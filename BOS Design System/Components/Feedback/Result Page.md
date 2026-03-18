# Result Page

> **Page:** ↪︎ Result page
> **Type:** Component — Feedback

---

## Overview

A centered full-page (or full-panel) result state displayed after a user action completes. Shows a large icon, a heading, an optional subtitle, and action buttons. The Error state includes an additional content box listing specific error details with inline links.

---

## States

| State | Icon | Description |
|---|---|---|
| `Info` | 80px blue info circle | Neutral result or in-progress |
| `Success` | 80px green check circle | Action completed successfully |
| `Error` | 80px red cancel circle | Action failed |

---

## Dimensions

| Property | Value |
|---|---|
| Container width | 600px |
| Icon | 80 × 80px |
| Gap (between elements) | 16px |
| Heading font | 22px/40px regular |
| Subtitle font | 14px/26px regular |

---

## Colors

| Element | Color |
|---|---|
| Background | `#fafafa` |
| Heading text | `#242424` |
| Subtitle text | `#242424` |
| Error content bg | `#fafafa` |
| Error content text | `rgba(0,0,0,0.85)` |
| Error subtext | `rgba(0,0,0,0.45)` |
| Error link | `#1890ff` |

---

## Icon Colors by State

| State | Icon token | Color |
|---|---|---|
| `Info` | `color/primary/500☆` | `#2d79c8` |
| `Success` | `color/primary/100` | `#d2e4f8` (circle fill) |
| `Error` | `color/error/500☆` | `#e62822` |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `result-page/background/default` | `#fafafa` | Page/content bg |
| `result-page/text/default` | `#242424` | Heading text |
| `color/primary/500☆` | `#2d79c8` | Info icon |
| `color/error/500☆` | `#e62822` | Error icon |

---

## Error Content Box

The Error state includes a content panel below the heading:

| Property | Value |
|---|---|
| Background | `#fafafa` |
| Padding | 24px vertical, 40px horizontal |
| Section heading | 16px/28px regular, `rgba(0,0,0,0.85)` |
| Error item icon | 16 × 16px close-circle, `#e62822` |
| Error item text | 14px/26px regular, `rgba(0,0,0,0.85)` |
| Error item link | 14px/26px regular, `#1890ff` |
| Gap between items | 14px |

---

## CSS Custom Properties

```css
:root {
  --result-bg:           #fafafa;
  --result-text:         #242424;
  --result-subtext:      rgba(0, 0, 0, 0.45);
  --result-error-text:   rgba(0, 0, 0, 0.85);
  --result-link:         #1890ff;
}
```

---

## CSS Classes

```css
/* ── Result wrapper ───────────────────────── */
.bos-result {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: var(--space-xl, 16px);
  width: 600px;
  font-family: var(--font-family);
  background-color: var(--result-bg, #fafafa);
}

/* ── Icon ─────────────────────────────────── */
.bos-result__icon {
  width: 80px;
  height: 80px;
  flex-shrink: 0;
}

/* ── Heading ──────────────────────────────── */
.bos-result__heading {
  font-size: 22px;
  line-height: 40px;
  font-weight: var(--font-weight-regular);
  color: var(--result-text, #242424);
  text-align: center;
  width: 100%;
}

/* ── Subtitle ─────────────────────────────── */
.bos-result__subtitle {
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--result-subtext, rgba(0,0,0,0.45));
  text-align: center;
  width: 100%;
}

/* ── Button group ─────────────────────────── */
.bos-result__actions {
  display: flex;
  align-items: center;
  gap: var(--space-m, 8px);
}

/* ── Error content box ────────────────────── */
.bos-result__error-content {
  width: 100%;
  background-color: var(--result-bg, #fafafa);
  padding: 24px 40px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.bos-result__error-heading {
  font-size: 16px;
  line-height: 28px;
  font-weight: var(--font-weight-regular);
  color: var(--result-error-text, rgba(0,0,0,0.85));
}

.bos-result__error-item {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: var(--text-body-size);
  line-height: var(--text-body-line-height);
}

.bos-result__error-item-icon {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  color: #e62822;
}

.bos-result__error-item-text {
  color: var(--result-error-text, rgba(0,0,0,0.85));
  white-space: nowrap;
}

.bos-result__error-item-link {
  color: var(--result-link, #1890ff);
  white-space: nowrap;
}
```

---

## HTML Usage

```html
<!-- Info state -->
<div class="bos-result">
  <img class="bos-result__icon" src="icon-info.svg" alt="" />
  <p class="bos-result__heading">Your operation has been executed</p>
  <p class="bos-result__subtitle">Order number: 2017182818828182881</p>
  <div class="bos-result__actions">
    <button class="bos-btn bos-btn--outline-secondary">Subtle</button>
    <button class="bos-btn bos-btn--primary">Main</button>
  </div>
</div>

<!-- Success state -->
<div class="bos-result">
  <img class="bos-result__icon" src="icon-success.svg" alt="" />
  <p class="bos-result__heading">ดำเนินการสำเร็จ</p>
  <p class="bos-result__subtitle">ข้อมูลของคุณถูกบันทึกเรียบร้อยแล้ว</p>
  <div class="bos-result__actions">
    <button class="bos-btn bos-btn--primary">กลับหน้าหลัก</button>
  </div>
</div>

<!-- Error state (with error content box) -->
<div class="bos-result">
  <img class="bos-result__icon" src="icon-error.svg" alt="" />
  <p class="bos-result__heading">Submission Failed</p>
  <p class="bos-result__subtitle">กรุณาตรวจสอบข้อมูลและลองอีกครั้ง</p>
  <div class="bos-result__error-content">
    <p class="bos-result__error-heading">The content you submitted has the following error:</p>
    <div class="bos-result__error-item">
      <svg class="bos-result__error-item-icon" width="16" height="16"><!-- close-circle --></svg>
      <span class="bos-result__error-item-text">Your account has been frozen.</span>
      <a class="bos-result__error-item-link" href="#">Thaw immediately ›</a>
    </div>
    <div class="bos-result__error-item">
      <svg class="bos-result__error-item-icon" width="16" height="16"><!-- close-circle --></svg>
      <span class="bos-result__error-item-text">Your account is not yet eligible to apply.</span>
      <a class="bos-result__error-item-link" href="#">Apply Unlock ›</a>
    </div>
  </div>
  <div class="bos-result__actions">
    <button class="bos-btn bos-btn--outline-secondary">Subtle</button>
    <button class="bos-btn bos-btn--primary">Main</button>
  </div>
</div>
```

---

## Usage Notes

- Container width is 600px — center it within the page or panel.
- The icon is always 80px; use filled circle icons (info, check, cancel) in the state's semantic color.
- The subtitle is optional — omit `bos-result__subtitle` when no additional context is needed.
- The error content box is unique to the Error state — do not use it in Info or Success states.
- Button group is centered; the subtle (outline) button is optional — use only when a secondary action exists.
