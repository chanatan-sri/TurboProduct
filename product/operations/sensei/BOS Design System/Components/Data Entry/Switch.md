# Switch

> **Page:** ↪︎ Switch
> **Type:** Component — Data Entry

---

## Overview

A toggle switch for binary on/off states. Supports Default and Small sizes, active/inactive states, disabled state, and an optional label. The thumb slides within the track to indicate state.

---

## Sizes

| Size | Track | Thumb |
|---|---|---|
| `Default` | 44 × 22px | 18px diameter |
| `Small` | 34 × 18px | 14px diameter |

---

## States

| State | Description |
|---|---|
| `Default` | Resting — inactive |
| `Active` | Toggled on |
| `Disabled` | Not interactive |

---

## Colors

| State | Track | Thumb |
|---|---|---|
| `Inactive` | `#e2e1e1` | `#fafafa` |
| `Active` | `#2d79c8` | `#fafafa` |
| `Disabled (inactive)` | `#e2e1e1` (opacity 50%) | `#fafafa` |
| `Disabled (active)` | `#2d79c8` (opacity 50%) | `#fafafa` |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `color/primary/500☆` | `#2d79c8` | Active track |
| `color/neutral/200` | `#e2e1e1` | Inactive track |
| `color/neutral/0-white` | `#fafafa` | Thumb |
| `color/neutral/900-black` | `#242424` | Label text |
| `color/neutral/300` | `#adaaaa` | Disabled label |

---

## CSS Custom Properties

```css
:root {
  --switch-track-active:   #2d79c8;
  --switch-track-inactive: #e2e1e1;
  --switch-thumb:          #fafafa;
  --switch-label:          #242424;
  --switch-label-disabled: #adaaaa;
}
```

---

## CSS Classes

```css
/* ── Wrapper ──────────────────────────────── */
.bos-switch {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-family: var(--font-family);
}

/* ── Track ────────────────────────────────── */
.bos-switch__track {
  position: relative;
  width: 44px;
  height: 22px;
  border-radius: 9999px;
  background-color: var(--switch-track-inactive, #e2e1e1);
  flex-shrink: 0;
  transition: background-color 0.2s;
}

/* ── Thumb ────────────────────────────────── */
.bos-switch__thumb {
  position: absolute;
  top: 2px;
  left: 2px;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background-color: var(--switch-thumb, #fafafa);
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.15);
  transition: transform 0.2s;
}

/* ── Active state ─────────────────────────── */
.bos-switch--active .bos-switch__track {
  background-color: var(--switch-track-active, #2d79c8);
}

.bos-switch--active .bos-switch__thumb {
  transform: translateX(22px);
}

/* ── Label ────────────────────────────────── */
.bos-switch__label {
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--switch-label, #242424);
}

/* ── Size — Small ─────────────────────────── */
.bos-switch--sm .bos-switch__track {
  width: 34px;
  height: 18px;
}

.bos-switch--sm .bos-switch__thumb {
  width: 14px;
  height: 14px;
}

.bos-switch--sm.bos-switch--active .bos-switch__thumb {
  transform: translateX(16px);
}

/* ── Disabled ─────────────────────────────── */
.bos-switch--disabled {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

.bos-switch--disabled .bos-switch__label {
  color: var(--switch-label-disabled, #adaaaa);
}
```

---

## HTML Usage

```html
<!-- Inactive (default) -->
<label class="bos-switch">
  <div class="bos-switch__track">
    <div class="bos-switch__thumb"></div>
  </div>
  <span class="bos-switch__label">Label</span>
</label>

<!-- Active -->
<label class="bos-switch bos-switch--active">
  <div class="bos-switch__track">
    <div class="bos-switch__thumb"></div>
  </div>
  <span class="bos-switch__label">เปิดใช้งาน</span>
</label>

<!-- Small — inactive -->
<label class="bos-switch bos-switch--sm">
  <div class="bos-switch__track">
    <div class="bos-switch__thumb"></div>
  </div>
  <span class="bos-switch__label">Label</span>
</label>

<!-- Small — active -->
<label class="bos-switch bos-switch--sm bos-switch--active">
  <div class="bos-switch__track">
    <div class="bos-switch__thumb"></div>
  </div>
  <span class="bos-switch__label">Label</span>
</label>

<!-- Disabled -->
<label class="bos-switch bos-switch--disabled">
  <div class="bos-switch__track">
    <div class="bos-switch__thumb"></div>
  </div>
  <span class="bos-switch__label">ไม่พร้อมใช้งาน</span>
</label>

<!-- Switch without label -->
<label class="bos-switch">
  <div class="bos-switch__track">
    <div class="bos-switch__thumb"></div>
  </div>
</label>
```

---

## Usage Notes

- The thumb travels from `left: 2px` (inactive) to `left: 24px` (active) for Default size.
- For Small size the thumb travels from `left: 2px` to `left: 18px`.
- Always use a `<label>` wrapper so clicking the label text also toggles the switch.
- Switch is pill-shaped — do not alter the `border-radius`.
