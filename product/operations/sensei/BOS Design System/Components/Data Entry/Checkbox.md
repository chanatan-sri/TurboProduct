# Checkbox

> **Page:** ↪︎ Check box
> **Type:** Component — Data Entry

---

## Overview

Checkboxes allow users to select one or more options from a set. Each checkbox has an optional label and help text, supports indeterminate state, and shows validation error styling.

---

## States

| State | Description |
|---|---|
| `Default` | Resting state |
| `Hover` | Mouse over |
| `Disabled` | Not interactive |
| `Prefill` | Pre-populated value |
| `Validated` | Error — form validation failed |

---

## Checked States

| Checked | Icon | Background | Border |
|---|---|---|---|
| `False` | None | `#fafafa` | 2px `#adaaaa` |
| `True` | White check ✓ | `#2d79c8` | None |
| `Indeterminate` | White dash — | `#2d79c8` | None |

---

## Validation (Error) State

| Element | Color |
|---|---|
| Background | `color/error/50` = `#fff2f0` |
| Icon / Stroke | `color/error/500☆` = `#e62822` |
| Help text | `#e62822` |

---

## Dimensions

| Property | Value |
|---|---|
| Checkbox size | 16 × 16px |
| Corner radius | 4px |
| Border (unchecked) | 2px solid |
| Label font | 14px/26px regular |
| Help text font | 12px/22px regular |
| Padding | 4px left/right |

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `checkbox/background/default` | `color/neutral/0-white` | `#fafafa` | Unchecked bg |
| `checkbox/background/active` | `color/primary/500☆` | `#2d79c8` | Checked bg |
| `checkbox/background/disabled` | `color/neutral/200` | `#e2e1e1` | Disabled bg |
| `checkbox/background/validated-default` | `color/error/50` | `#fff2f0` | Error bg |
| `checkbox/stroke/default` | `color/neutral/300` | `#adaaaa` | Unchecked border |
| `checkbox/stroke/hover` | `color/primary/400` | `#4a8fd4` | Hover border |
| `checkbox/stroke/disabled` | `color/error/500☆` | `#e62822` | Disabled (uses error) |
| `checkbox/text/value-default` | `color/neutral/900-black` | `#242424` | Label default |
| `checkbox/text/value-active` | `color/neutral/900-black` | `#242424` | Label checked |
| `checkbox/text/value-disabled` | `color/neutral/300` | `#adaaaa` | Label disabled |
| `checkbox/text/help-text-default` | `color/neutral/600` | `#625e5e` | Help text |
| `checkbox/text/help-text-validated` | `color/neutral/300` | `#adaaaa` | Help text disabled |

---

## CSS Custom Properties

```css
:root {
  --checkbox-bg-default:    #fafafa;
  --checkbox-bg-active:     #2d79c8;
  --checkbox-bg-disabled:   #e2e1e1;
  --checkbox-bg-error:      #fff2f0;
  --checkbox-stroke:        #adaaaa;
  --checkbox-stroke-hover:  #4a8fd4;
  --checkbox-stroke-error:  #e62822;
  --checkbox-text:          #242424;
  --checkbox-text-disabled: #adaaaa;
  --checkbox-help:          #625e5e;
  --checkbox-help-error:    #e62822;
}
```

---

## CSS Classes

```css
/* ── Trigger wrapper ────────────────────────── */
.bos-checkbox {
  display: inline-flex;
  flex-direction: column;
  align-items: flex-start;
  padding: 0 var(--space-xs, 4px);
  cursor: pointer;
  font-family: var(--font-family);
}

/* ── Checkbox + label row ────────────────────── */
.bos-checkbox__row {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* ── The box itself ──────────────────────────── */
.bos-checkbox__box {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  border-radius: var(--corner-xs, 4px);
  border: 2px solid var(--checkbox-stroke, #adaaaa);
  background-color: var(--checkbox-bg-default, #fafafa);
  display: flex;
  align-items: center;
  justify-content: center;
}

.bos-checkbox__box:hover {
  border-color: var(--checkbox-stroke-hover, #4a8fd4);
}

/* Checked state */
.bos-checkbox input:checked ~ .bos-checkbox__row .bos-checkbox__box,
.bos-checkbox__box--checked {
  background-color: var(--checkbox-bg-active, #2d79c8);
  border-color: var(--checkbox-bg-active, #2d79c8);
}

/* Indeterminate */
.bos-checkbox__box--indeterminate {
  background-color: var(--checkbox-bg-active, #2d79c8);
  border-color: var(--checkbox-bg-active, #2d79c8);
}

/* Disabled */
.bos-checkbox--disabled {
  cursor: not-allowed;
  pointer-events: none;
}
.bos-checkbox--disabled .bos-checkbox__box {
  background-color: var(--checkbox-bg-disabled, #e2e1e1);
  border-color: var(--checkbox-bg-disabled, #e2e1e1);
}
.bos-checkbox--disabled .bos-checkbox__label {
  color: var(--checkbox-text-disabled, #adaaaa);
}

/* Validated (error) */
.bos-checkbox--error .bos-checkbox__box {
  background-color: var(--checkbox-bg-error, #fff2f0);
  border-color: var(--checkbox-stroke-error, #e62822);
}
.bos-checkbox--error .bos-checkbox__help {
  color: var(--checkbox-help-error, #e62822);
}

/* ── Check icon ─────────────────────────────── */
.bos-checkbox__icon {
  width: 10px;
  height: 10px;
  color: #ffffff;
}

/* ── Label ───────────────────────────────────── */
.bos-checkbox__label {
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--checkbox-text, #242424);
  padding: 4px 0;
}

/* ── Help text ───────────────────────────────── */
.bos-checkbox__help {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding-bottom: 4px;
}

.bos-checkbox__help-spacer {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

.bos-checkbox__help p {
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  font-weight: var(--font-weight-regular);
  color: var(--checkbox-help, #625e5e);
}
```

---

## HTML Usage

```html
<!-- Unchecked with label + help text -->
<label class="bos-checkbox">
  <div class="bos-checkbox__row">
    <div class="bos-checkbox__box"></div>
    <span class="bos-checkbox__label">Label</span>
  </div>
  <div class="bos-checkbox__help">
    <div class="bos-checkbox__help-spacer"></div>
    <p>Help text</p>
  </div>
</label>

<!-- Checked -->
<label class="bos-checkbox">
  <div class="bos-checkbox__row">
    <div class="bos-checkbox__box bos-checkbox__box--checked">
      <svg class="bos-checkbox__icon" width="10" height="10"><!-- check --></svg>
    </div>
    <span class="bos-checkbox__label">Label</span>
  </div>
</label>

<!-- Indeterminate -->
<label class="bos-checkbox">
  <div class="bos-checkbox__row">
    <div class="bos-checkbox__box bos-checkbox__box--indeterminate">
      <svg class="bos-checkbox__icon" width="10" height="2"><!-- dash --></svg>
    </div>
    <span class="bos-checkbox__label">Select all</span>
  </div>
</label>

<!-- Disabled unchecked -->
<label class="bos-checkbox bos-checkbox--disabled">
  <div class="bos-checkbox__row">
    <div class="bos-checkbox__box"></div>
    <span class="bos-checkbox__label">Disabled</span>
  </div>
</label>

<!-- Validated (error) -->
<label class="bos-checkbox bos-checkbox--error">
  <div class="bos-checkbox__row">
    <div class="bos-checkbox__box"></div>
    <span class="bos-checkbox__label">Required field</span>
  </div>
  <div class="bos-checkbox__help">
    <div class="bos-checkbox__help-spacer"></div>
    <p>กรุณาเลือกอย่างน้อย 1 ข้อ</p>
  </div>
</label>

<!-- Checkbox group -->
<div style="display: flex; flex-direction: column; gap: 0;">
  <label class="bos-checkbox">
    <div class="bos-checkbox__row">
      <div class="bos-checkbox__box bos-checkbox__box--checked">
        <svg class="bos-checkbox__icon"><!-- check --></svg>
      </div>
      <span class="bos-checkbox__label">Option 1</span>
    </div>
  </label>
  <label class="bos-checkbox">
    <div class="bos-checkbox__row">
      <div class="bos-checkbox__box"></div>
      <span class="bos-checkbox__label">Option 2</span>
    </div>
  </label>
</div>
```

---

## Usage Notes

- Checkbox size is fixed at **16 × 16px** — do not resize the box.
- Always align the help text spacer (16px) under the checkbox box for proper indent.
- Use `indeterminate` state for "select all" parent checkboxes in a partially-selected group.
- Validation error changes both the box styling and the help text color to red.
