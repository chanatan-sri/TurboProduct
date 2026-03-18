# Input

> **Page:** ↪︎ Input
> **Type:** Component — Data Entry

---

## Overview

A full-featured text input field. Supports label, help text, action link, prefix/suffix icons, clear button, unit suffix, character count, and multiple states. Also includes a **Multiline** (textarea) variant with optional toolbar.

---

## Sizes

| Size | Height (input row) | Total height (with label) |
|---|---|---|
| `Default` | 30px | 62px |
| `Large` | 38px | 70px |

---

## States

| State | Border | Background |
|---|---|---|
| `Default` | `#e2e1e1` | `#fafafa` |
| `Hover` | `#5494db` | `#fafafa` |
| `Active` (focus) | `#2d79c8` | `#fafafa` |
| `Have value` | `#e2e1e1` | `#fafafa` |
| `Disabled` | `#e2e1e1` | `#e2e1e1` |
| `Prefill` | `#e2e1e1` | `#e2e1e1` |
| `Validated` (error) | `#e62822` | `#fff2f0` |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `input/background/default` | `#fafafa` | Input bg |
| `input/background/hover` | `#fafafa` | Hover bg |
| `input/background/disabled` | `#e2e1e1` | Disabled bg |
| `input/background/validated` | `#fff2f0` | Error bg |
| `input/stroke/action-border-default` | `#e2e1e1` | Default border |
| `input/stroke/action-border-hover` | `#5494db` | Hover border |
| `input/stroke/action-border-active` | `#2d79c8` | Active border |
| `input/stroke/validated` | `#e62822` | Error border |
| `input/text/label-text-default` | `#322f2f` | Label |
| `input/text/value-text-have-value` | `#242424` | Value text |
| `input/text/value-text-disabled` | `#adaaaa` | Disabled/unit text |
| `input/text/help-text-default` | `#625e5e` | Help text |
| `input/text/action-text-default` | `#2d79c8` | Action link |

---

## CSS Custom Properties

```css
:root {
  --input-bg:              #fafafa;
  --input-bg-disabled:     #e2e1e1;
  --input-bg-error:        #fff2f0;
  --input-border:          #e2e1e1;
  --input-border-hover:    #5494db;
  --input-border-active:   #2d79c8;
  --input-border-error:    #e62822;
  --input-text:            #242424;
  --input-label:           #322f2f;
  --input-placeholder:     #adaaaa;
  --input-help:            #625e5e;
  --input-action:          #2d79c8;
}
```

---

## CSS Classes

```css
/* ── Wrapper ─────────────────────────────────── */
.bos-input {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-100, 4px);
  font-family: var(--font-family);
  width: 100%;
}

/* ── Label row ───────────────────────────────── */
.bos-input__label-row {
  display: flex;
  align-items: baseline;
  gap: 8px;
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
}

.bos-input__label {
  flex: 1;
  color: var(--input-label, #322f2f);
  font-weight: var(--font-weight-regular);
}

.bos-input__optional {
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  color: var(--input-placeholder, #adaaaa);
}

.bos-input__action {
  font-size: var(--text-body-size);
  color: var(--input-action, #2d79c8);
  cursor: pointer;
}

/* ── Input field ─────────────────────────────── */
.bos-input__field {
  display: flex;
  align-items: center;
  gap: var(--p-space-100, 4px);
  padding: var(--space-2xs, 2px) var(--p-space-300, 12px);
  background-color: var(--input-bg, #fafafa);
  border: 1px solid var(--input-border, #e2e1e1);
  border-radius: var(--corner-s, 6px);
  height: 30px;
}

.bos-input__field:hover {
  border-color: var(--input-border-hover, #5494db);
}

.bos-input__field:focus-within {
  border-color: var(--input-border-active, #2d79c8);
}

/* ── Input text ──────────────────────────────── */
.bos-input__text {
  flex: 1;
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--input-text, #242424);
  background: transparent;
  border: none;
  outline: none;
  padding-bottom: var(--space-2xs, 2px);
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.bos-input__text::placeholder {
  color: var(--input-placeholder, #adaaaa);
}

/* ── Icons & buttons ─────────────────────────── */
.bos-input__icon {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  color: var(--input-placeholder, #adaaaa);
}

.bos-input__clear {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  cursor: pointer;
  color: var(--input-placeholder, #adaaaa);
}

.bos-input__unit {
  font-size: var(--text-body-size);
  color: var(--input-placeholder, #adaaaa);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* ── Bottom row (help text + character count) ── */
.bos-input__bottom {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 4px;
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  color: var(--input-help, #625e5e);
}

.bos-input__help { flex: 1; }
.bos-input__count { white-space: nowrap; }

/* ── Size — Large ────────────────────────────── */
.bos-input--lg .bos-input__field { height: 38px; }

/* ── States ──────────────────────────────────── */
.bos-input--disabled .bos-input__field {
  background-color: var(--input-bg-disabled, #e2e1e1);
  border-color: var(--input-border, #e2e1e1);
  cursor: not-allowed;
}

.bos-input--disabled .bos-input__text {
  color: var(--input-placeholder, #adaaaa);
}

.bos-input--error .bos-input__field {
  background-color: var(--input-bg-error, #fff2f0);
  border-color: var(--input-border-error, #e62822);
}

.bos-input--error .bos-input__help {
  color: var(--input-border-error, #e62822);
}

/* ── Multiline (textarea) ────────────────────── */
.bos-input--multiline .bos-input__field {
  height: auto;
  min-height: 80px;
  align-items: flex-start;
  padding-top: var(--space-xs, 4px);
  padding-bottom: var(--space-xs, 4px);
}

.bos-input--multiline .bos-input__text {
  resize: vertical;
  white-space: normal;
  overflow: auto;
  text-overflow: unset;
}
```

---

## HTML Usage

```html
<!-- Default input with label -->
<div class="bos-input">
  <div class="bos-input__label-row">
    <span class="bos-input__label">Label</span>
  </div>
  <div class="bos-input__field">
    <svg class="bos-input__icon" width="16" height="16"><!-- prefix icon --></svg>
    <input class="bos-input__text" type="text" placeholder="Placeholder" />
    <svg class="bos-input__icon" width="16" height="16"><!-- dropdown --></svg>
    <svg class="bos-input__clear" width="16" height="16"><!-- × --></svg>
  </div>
</div>

<!-- With help text + optional label + action link -->
<div class="bos-input">
  <div class="bos-input__label-row">
    <span class="bos-input__label">ชื่อ <span class="bos-input__optional">(ถ้ามี)</span></span>
    <span class="bos-input__action">แก้ไข</span>
  </div>
  <div class="bos-input__field">
    <input class="bos-input__text" type="text" value="value" />
  </div>
  <div class="bos-input__bottom">
    <span class="bos-input__help">Help text</span>
    <span class="bos-input__count">0/100</span>
  </div>
</div>

<!-- With unit suffix -->
<div class="bos-input">
  <div class="bos-input__label-row">
    <span class="bos-input__label">ระยะทาง</span>
  </div>
  <div class="bos-input__field">
    <input class="bos-input__text" type="text" placeholder="0" />
    <span class="bos-input__unit">km</span>
  </div>
</div>

<!-- Disabled -->
<div class="bos-input bos-input--disabled">
  <div class="bos-input__label-row">
    <span class="bos-input__label">Label</span>
  </div>
  <div class="bos-input__field">
    <input class="bos-input__text" type="text" disabled />
  </div>
</div>

<!-- Error / Validated -->
<div class="bos-input bos-input--error">
  <div class="bos-input__label-row">
    <span class="bos-input__label">Label</span>
  </div>
  <div class="bos-input__field">
    <input class="bos-input__text" type="text" />
  </div>
  <div class="bos-input__bottom">
    <span class="bos-input__help">กรุณากรอกข้อมูล</span>
  </div>
</div>

<!-- Multiline (textarea) -->
<div class="bos-input bos-input--multiline">
  <div class="bos-input__label-row">
    <span class="bos-input__label">หมายเหตุ</span>
  </div>
  <div class="bos-input__field">
    <textarea class="bos-input__text" placeholder="พิมพ์ที่นี่..."></textarea>
  </div>
</div>
```

---

## Usage Notes

- Total component height (including label) is **62px** (Default) or **70px** (Large).
- The prefix icon slot is 16×16px — use only a single icon.
- The clear (×) button appears only when the field has a value.
- Character count (`0/100`) appears bottom-right when `limitCharacter` is enabled.
- Multiline inputs expand vertically — no fixed height constraint.
- Do not change corner radius from 6px.
