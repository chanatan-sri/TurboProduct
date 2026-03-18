# Radio

> **Page:** ↪︎ Radio box
> **Type:** Component — Data Entry

---

## Overview

Radio buttons allow users to select exactly one option from a set. Each radio has an optional label and help text. They support checked/unchecked, hover, disabled, prefill, and validated (error) states.

---

## States

| State | Description |
|---|---|
| `Default` | Resting |
| `Hover` | Mouse over |
| `Disabled` | Not interactive |
| `Prefill` | Pre-populated |
| `Validated` | Error — form validation failed |

---

## Checked States

| Checked | Icon | Color |
|---|---|---|
| `False` | Empty circle outline | `#adaaaa` (neutral/300) |
| `True` | Filled circle (inner dot) | `#2d79c8` (primary/500☆) |
| `Disabled=True` | Filled/empty — reduced opacity | `#e2e1e1` (neutral/200) |
| `Validated (error)` | Circle outline | `#e62822` (error/500☆) |

---

## Dimensions

| Property | Value |
|---|---|
| Icon container | 24 × 24px |
| Radio circle | 14px diameter |
| Label font | 14px/26px regular |
| Help text font | 12px/22px regular |

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `radio-box/background/default` | `color/neutral/0-white` | `#fafafa` | Unchecked bg |
| `radio-box/background/active` | `color/primary/500☆` | `#2d79c8` | Checked fill |
| `radio-box/background/disabled` | `color/neutral/200` | `#e2e1e1` | Disabled |
| `radio-box/background/validated` | `color/error/50` | `#fff2f0` | Error bg |
| `radio-box/stroke/default` | `color/neutral/300` | `#adaaaa` | Unchecked border |
| `radio-box/stroke/hover` | `color/primary/400` | `#4a8fd4` | Hover border |
| `radio-box/stroke/validated` | `color/error/500☆` | `#e62822` | Error border |
| `radio-box/text/label-inactive` | `color/neutral/900-black` | `#242424` | Label text |
| `radio-box/text/label-disabled` | `color/neutral/300` | `#adaaaa` | Disabled label |
| `radio-box/text/help-text-inactive` | `color/neutral/600` | `#625e5e` | Help text |
| `radio-box/text/error-message` | `color/error/500☆` | `#e62822` | Error message |

---

## CSS Custom Properties

```css
:root {
  --radio-color:            #adaaaa;
  --radio-color-active:     #2d79c8;
  --radio-color-hover:      #4a8fd4;
  --radio-color-disabled:   #e2e1e1;
  --radio-color-error:      #e62822;
  --radio-label:            #242424;
  --radio-label-disabled:   #adaaaa;
  --radio-help:             #625e5e;
}
```

---

## CSS Classes

```css
/* ── Radio item ──────────────────────────────── */
.bos-radio {
  display: inline-flex;
  flex-direction: column;
  gap: 0;
  cursor: pointer;
  font-family: var(--font-family);
}

/* ── Label row ───────────────────────────────── */
.bos-radio__row {
  display: flex;
  align-items: center;
  gap: 4px;
}

/* ── Icon container ──────────────────────────── */
.bos-radio__icon {
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

/* ── The radio circle ────────────────────────── */
.bos-radio__circle {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  border: 2px solid var(--radio-color, #adaaaa);
  background: #fafafa;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: border-color 0.15s;
}

.bos-radio__row:hover .bos-radio__circle {
  border-color: var(--radio-color-hover, #4a8fd4);
}

/* Checked */
.bos-radio__circle--checked {
  border-color: var(--radio-color-active, #2d79c8);
  background: var(--radio-color-active, #2d79c8);
}

.bos-radio__dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #ffffff;
}

/* ── Label ───────────────────────────────────── */
.bos-radio__label {
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--radio-label, #242424);
}

/* ── Help text ───────────────────────────────── */
.bos-radio__help {
  display: flex;
  align-items: flex-start;
  gap: var(--space-xs, 4px);
}

.bos-radio__help-spacer { width: 24px; flex-shrink: 0; }

.bos-radio__help p {
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  color: var(--radio-help, #625e5e);
}

/* ── Disabled ────────────────────────────────── */
.bos-radio--disabled {
  cursor: not-allowed;
  pointer-events: none;
}

.bos-radio--disabled .bos-radio__circle {
  border-color: var(--radio-color-disabled, #e2e1e1);
}

.bos-radio--disabled .bos-radio__label {
  color: var(--radio-label-disabled, #adaaaa);
}

/* ── Error / Validated ───────────────────────── */
.bos-radio--error .bos-radio__circle {
  border-color: var(--radio-color-error, #e62822);
}

.bos-radio--error .bos-radio__help p {
  color: var(--radio-color-error, #e62822);
}

/* ── Radio group ─────────────────────────────── */
.bos-radio-group {
  display: flex;
  flex-direction: column;
  gap: 0;
}
```

---

## HTML Usage

```html
<!-- Unchecked radio -->
<label class="bos-radio">
  <div class="bos-radio__row">
    <div class="bos-radio__icon">
      <div class="bos-radio__circle"></div>
    </div>
    <span class="bos-radio__label">Label</span>
  </div>
  <div class="bos-radio__help">
    <div class="bos-radio__help-spacer"></div>
    <p>Help text</p>
  </div>
</label>

<!-- Checked radio -->
<label class="bos-radio">
  <div class="bos-radio__row">
    <div class="bos-radio__icon">
      <div class="bos-radio__circle bos-radio__circle--checked">
        <div class="bos-radio__dot"></div>
      </div>
    </div>
    <span class="bos-radio__label">Selected option</span>
  </div>
</label>

<!-- Disabled radio -->
<label class="bos-radio bos-radio--disabled">
  <div class="bos-radio__row">
    <div class="bos-radio__icon">
      <div class="bos-radio__circle"></div>
    </div>
    <span class="bos-radio__label">Disabled</span>
  </div>
</label>

<!-- Error / validated -->
<label class="bos-radio bos-radio--error">
  <div class="bos-radio__row">
    <div class="bos-radio__icon">
      <div class="bos-radio__circle"></div>
    </div>
    <span class="bos-radio__label">Required</span>
  </div>
  <div class="bos-radio__help">
    <div class="bos-radio__help-spacer"></div>
    <p>กรุณาเลือก 1 ตัวเลือก</p>
  </div>
</label>

<!-- Radio group -->
<div class="bos-radio-group" role="radiogroup">
  <label class="bos-radio">
    <div class="bos-radio__row">
      <div class="bos-radio__icon">
        <div class="bos-radio__circle bos-radio__circle--checked">
          <div class="bos-radio__dot"></div>
        </div>
      </div>
      <span class="bos-radio__label">Option 1</span>
    </div>
  </label>
  <label class="bos-radio">
    <div class="bos-radio__row">
      <div class="bos-radio__icon">
        <div class="bos-radio__circle"></div>
      </div>
      <span class="bos-radio__label">Option 2</span>
    </div>
  </label>
</div>
```

---

## Usage Notes

- Radio buttons are always used in groups — only one can be selected at a time.
- The inner dot is 6px white circle, centered inside the 14px circle.
- Help text spacer (24px) aligns the text under the radio circle.
- Use `bos-radio-group` to wrap sets of options.
