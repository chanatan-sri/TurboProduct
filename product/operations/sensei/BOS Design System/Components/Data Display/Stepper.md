# Stepper

> **Page:** ↪︎ Stepper
> **Type:** Component — Data Display

---

## Overview

A step-by-step progress indicator. Shows which steps are complete, active, or pending in a multi-step workflow.

---

## Sizes

| Size | Icon | Height |
|---|---|---|
| Default | 24 × 24px | ~32px per step |
| Small | 20 × 20px | ~28px per step |

---

## States

| State | Description |
|---|---|
| `Default` | Resting state |
| `Hover` | Mouse over — background tint appears |
| `Disabled` | Not interactive — reduced opacity |

---

## Validation

| Validation | Icon | Text Color |
|---|---|---|
| `Incomplete` | Circle / outline check | `#625e5e` (subtle) |
| `Complete` | Filled check ✓ | `#242424` (default) |
| `Error` | Alert / X icon | `#e62822` (danger) |

---

## Icon Position

| Position | Description |
|---|---|
| `Leading` | Icon before the label |
| `Trailing` | Icon after the label |

---

## With / Without Number

Steps can show a numeric sequence number inside the icon circle (e.g. `1`, `2`, `3`) or use only icons.

---

## Design Tokens (from Figma)

| Token | Hex | Usage |
|---|---|---|
| `stepper/background/default` | `transparent` | Step item bg (no fill) |
| `stepper/text/incomplete-default` | `#625e5e` | Incomplete step label |
| `stepper/text/complete-default` | `#242424` | Complete step label |
| `stepper/text/error` | `#e62822` | Error step label |
| `stepper/icon/incomplete` | `#adaaaa` | Incomplete icon/circle color |
| `stepper/icon/complete` | `#2d79c8` | Complete icon color |
| `stepper/icon/error` | `#e62822` | Error icon color |
| Connector line | `#e2e1e1` | Line between steps |
| Connector active | `#2d79c8` | Line after completed step |

---

## CSS Custom Properties

```css
:root {
  --stepper-text-incomplete: #625e5e;
  --stepper-text-complete:   #242424;
  --stepper-text-error:      #e62822;
  --stepper-icon-incomplete: #adaaaa;
  --stepper-icon-complete:   #2d79c8;
  --stepper-icon-error:      #e62822;
  --stepper-connector:       #e2e1e1;
  --stepper-connector-done:  #2d79c8;
  --stepper-hover-bg:        #f1f0f0;
}
```

---

## CSS Classes

```css
/* ── Stepper container ──────────────────────────── */
.bos-stepper {
  display: flex;
  flex-direction: column;
  gap: 0;
  font-family: var(--font-family);
}

/* Horizontal variant */
.bos-stepper--horizontal {
  flex-direction: row;
  align-items: center;
}

/* ── Step item ──────────────────────────────────── */
.bos-stepper__step {
  display: flex;
  align-items: flex-start;
  gap: 4px;
  padding: var(--space-xs, 4px) var(--space-m, 8px) var(--space-xs, 4px) var(--space-s, 6px);
  background-color: transparent;
  border-radius: var(--corner-xs, 4px);
  cursor: pointer;
}

.bos-stepper__step:hover {
  background-color: var(--stepper-hover-bg, #f1f0f0);
}

.bos-stepper__step--disabled {
  opacity: 0.4;
  cursor: not-allowed;
  pointer-events: none;
}

/* ── Sizes ──────────────────────────────────────── */
.bos-stepper__step--default .bos-stepper__icon { width: 24px; height: 24px; }
.bos-stepper__step--sm      .bos-stepper__icon { width: 20px; height: 20px; }

/* ── Icon ───────────────────────────────────────── */
.bos-stepper__icon {
  width: 24px;
  height: 24px;
  flex-shrink: 0;
  color: var(--stepper-icon-incomplete, #adaaaa);
}

/* Validation modifiers */
.bos-stepper__step--complete .bos-stepper__icon  { color: var(--stepper-icon-complete, #2d79c8); }
.bos-stepper__step--error    .bos-stepper__icon  { color: var(--stepper-icon-error, #e62822); }

/* ── Label ──────────────────────────────────────── */
.bos-stepper__label {
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--stepper-text-incomplete, #625e5e);
  white-space: nowrap;
}

.bos-stepper__step--complete .bos-stepper__label { color: var(--stepper-text-complete, #242424); }
.bos-stepper__step--error    .bos-stepper__label { color: var(--stepper-text-error, #e62822); }

/* ── Connector line ─────────────────────────────── */
.bos-stepper__connector {
  width: 1px;
  flex: 1;
  min-height: 16px;
  background-color: var(--stepper-connector, #e2e1e1);
  margin-left: 18px;  /* aligns under icon center */
}

.bos-stepper__connector--done {
  background-color: var(--stepper-connector-done, #2d79c8);
}

/* Horizontal connector */
.bos-stepper--horizontal .bos-stepper__connector {
  width: 40px;
  height: 1px;
  min-height: unset;
  margin-left: 0;
}
```

---

## HTML Usage

```html
<!-- Vertical stepper -->
<div class="bos-stepper">
  <!-- Step 1 — complete -->
  <div class="bos-stepper__step bos-stepper__step--default bos-stepper__step--complete">
    <svg class="bos-stepper__icon" width="24" height="24"><!-- check icon --></svg>
    <span class="bos-stepper__label">Step 1 — Completed</span>
  </div>

  <div class="bos-stepper__connector bos-stepper__connector--done"></div>

  <!-- Step 2 — incomplete (active) -->
  <div class="bos-stepper__step bos-stepper__step--default">
    <svg class="bos-stepper__icon" width="24" height="24"><!-- circle icon --></svg>
    <span class="bos-stepper__label">Step 2 — In Progress</span>
  </div>

  <div class="bos-stepper__connector"></div>

  <!-- Step 3 — error -->
  <div class="bos-stepper__step bos-stepper__step--default bos-stepper__step--error">
    <svg class="bos-stepper__icon" width="24" height="24"><!-- alert icon --></svg>
    <span class="bos-stepper__label">Step 3 — Error</span>
  </div>

  <div class="bos-stepper__connector"></div>

  <!-- Step 4 — disabled -->
  <div class="bos-stepper__step bos-stepper__step--default bos-stepper__step--disabled">
    <svg class="bos-stepper__icon" width="24" height="24"><!-- circle icon --></svg>
    <span class="bos-stepper__label">Step 4 — Pending</span>
  </div>
</div>
```

---

## Usage Notes

- Use in multi-step forms, onboarding flows, and approval workflows.
- Connector lines are optional — omit for standalone step lists.
- Small size (`bos-stepper__step--sm`) suits compact sidebars and panels.
- The `Trailing` icon variant places the icon after the label — useful for right-aligned layouts.
