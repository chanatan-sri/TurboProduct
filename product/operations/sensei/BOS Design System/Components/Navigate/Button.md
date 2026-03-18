# Button

> **Page:** ↪︎ Button
> **Type:** Component — Navigate

---

## Overview

Buttons trigger actions. BOS provides 5 visual types × 3 sizes × 4 states, with destructive variants and icon-only button variants.

---

## Types

| Type | Description |
|---|---|
| `Fill` | Primary action — solid filled background |
| `Tonal` | Secondary action — tinted background |
| `Outline Primary` | Bordered with primary color stroke |
| `Outline Secondary` | Bordered with neutral stroke |
| `Text` | No background or border — text only |
| `Icon Fill` | Icon-only, filled style |
| `Icon Tonal` | Icon-only, tonal style |
| `Icon Outline Primary` | Icon-only, primary outline |
| `Icon Outline Secondary` | Icon-only, secondary outline |
| `Icon Text` | Icon-only, text style |

---

## Sizes

| Size | Height | Min-width | Padding (H) |
|---|---|---|---|
| Small | 24px | 80px | `var(--space-l, 12px)` |
| Default | 32px | 80px | `var(--space-l, 12px)` |
| Large | 40px | 80px (84px actual) | `var(--space-l, 12px)` |

---

## States

| State | Description |
|---|---|
| Default | Normal resting state |
| Hover | Cursor over button |
| Pressed | Click/tap in progress |
| Disabled | Not interactive |

---

## Destructive Variant

All types support a `🚨 Destructive` flag. Destructive buttons use red/error color tokens instead of primary.

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `button/fill/background/primary-default` | `#2d79c8` | Fill bg — default |
| `button/fill/text/primary-default` | `#fafafa` | Fill text — default |

> **Destructive color** uses the BOS **Red** color palette (not Turbo Pink). Fill destructive bg ≈ `color/red/600`.

---

## State Behaviour (from Figma)

| Type | Default | Hover | Pressed | Disabled |
|---|---|---|---|---|
| Fill | solid blue bg | **lighter** blue bg | **darker** blue bg | gray bg + gray text |
| Tonal | light blue bg + blue text | more saturated bg | more saturated bg | gray bg + gray text |
| Outline Primary | transparent + blue border + blue text | light blue tint bg | more blue tint | gray border + gray text |
| Outline Secondary | transparent + gray border + dark text | **changes to blue** (border + text) + blue tint bg | more blue tint | gray bg + gray text |
| Text | blue text only | **gray bg box** appears | darker gray bg | faded gray text only |

---

## Corner Radius

```css
border-radius: var(--corner-m, 8px);  /* All button types */
```

---

## CSS Custom Properties

```css
:root {
  /* Fill */
  --btn-fill-bg:            #2d79c8;
  --btn-fill-hover-bg:      #4a8fd4;   /* lighter on hover */
  --btn-fill-pressed-bg:    #1a5da0;   /* darker on press */
  --btn-fill-text:          #fafafa;

  /* Tonal */
  --btn-tonal-bg:           #d2e4f8;
  --btn-tonal-hover-bg:     #b9d5f3;
  --btn-tonal-pressed-bg:   #9fc5ef;
  --btn-tonal-text:         #2d79c8;

  /* Outline Primary */
  --btn-outline-primary-border: #2d79c8;
  --btn-outline-primary-text:   #2d79c8;
  --btn-outline-primary-hover-bg: #e8f2fb;
  --btn-outline-primary-pressed-bg: #c5ddf5;

  /* Outline Secondary — switches to primary color on hover */
  --btn-outline-secondary-border: #e2e1e1;
  --btn-outline-secondary-text:   #242424;

  /* Text — gray box appears on hover */
  --btn-text-color:         #2d79c8;
  --btn-text-hover-bg:      #f1f0f0;
  --btn-text-pressed-bg:    #e2e1e1;

  /* Disabled (all types) */
  --btn-disabled-bg:        #f1f0f0;
  --btn-disabled-text:      #adaaaa;
  --btn-disabled-border:    #e2e1e1;

  /* Destructive — BOS Red palette (🌈 Color page › Red) */
  --btn-destructive-fill-bg:     #e62822;  /* color/red/500 — Error/Danger token */
  --btn-destructive-fill-text:   #fafafa;
  --btn-destructive-tonal-bg:    #fff2f0;  /* color/red/50 */
  --btn-destructive-tonal-text:  #e62822;  /* color/red/500 */
  --btn-destructive-color:       #e62822;  /* color/red/500 */
}
```

---

## CSS Classes

```css
/* ── Base ──────────────────────────────────────── */
.bos-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-m, 8px);
  min-width: 80px;
  padding: 0 var(--space-l, 12px);
  height: 32px;                       /* Default size */
  border-radius: var(--corner-m, 8px);
  border: none;
  font-family: var(--font-family);
  font-size: var(--text-body-size);   /* 14px */
  line-height: var(--text-body-line-height);
  font-weight: var(--font-weight-regular);
  white-space: nowrap;
  cursor: pointer;
  text-align: center;
  text-decoration: none;
  transition: background-color 0.15s, border-color 0.15s, color 0.15s;
}

/* ── Sizes ──────────────────────────────────────── */
.bos-btn--sm    { height: 24px; }
.bos-btn--lg    { height: 40px; min-width: 84px; }

/* ── Fill ──────────────────────────────────────── */
.bos-btn--fill {
  background-color: var(--btn-fill-bg, #2d79c8);
  color: var(--btn-fill-text, #fafafa);
}
.bos-btn--fill:hover   { background-color: var(--btn-fill-hover-bg, #4a8fd4); }   /* lighter */
.bos-btn--fill:active  { background-color: var(--btn-fill-pressed-bg, #1a5da0); } /* darker */

/* ── Tonal ─────────────────────────────────────── */
.bos-btn--tonal {
  background-color: var(--btn-tonal-bg, #d2e4f8);
  color: var(--btn-tonal-text, #2d79c8);
}
.bos-btn--tonal:hover  { background-color: var(--btn-tonal-hover-bg, #b9d5f3); }
.bos-btn--tonal:active { background-color: var(--btn-tonal-pressed-bg, #9fc5ef); }

/* ── Outline Primary ────────────────────────────── */
.bos-btn--outline-primary {
  background-color: transparent;
  border: 1px solid var(--btn-outline-primary-border, #2d79c8);
  color: var(--btn-outline-primary-text, #2d79c8);
}
.bos-btn--outline-primary:hover  { background-color: var(--btn-outline-primary-hover-bg, #e8f2fb); }
.bos-btn--outline-primary:active { background-color: var(--btn-outline-primary-pressed-bg, #c5ddf5); }

/* ── Outline Secondary ──────────────────────────── */
/* Default: gray border + dark text */
/* Hover/Pressed: switches fully to primary (blue border + blue text) */
.bos-btn--outline-secondary {
  background-color: transparent;
  border: 1px solid var(--btn-outline-secondary-border, #e2e1e1);
  color: var(--btn-outline-secondary-text, #242424);
}
.bos-btn--outline-secondary:hover {
  background-color: var(--btn-outline-primary-hover-bg, #e8f2fb);
  border-color: var(--btn-outline-primary-border, #2d79c8);
  color: var(--btn-outline-primary-text, #2d79c8);
}
.bos-btn--outline-secondary:active {
  background-color: var(--btn-outline-primary-pressed-bg, #c5ddf5);
  border-color: var(--btn-outline-primary-border, #2d79c8);
  color: var(--btn-outline-primary-text, #2d79c8);
}

/* ── Text ───────────────────────────────────────── */
/* Hover/Pressed: gray background box appears — no underline */
.bos-btn--text {
  background-color: transparent;
  color: var(--btn-text-color, #2d79c8);
  min-width: 0;
  border-radius: var(--corner-m, 8px);
}
.bos-btn--text:hover  { background-color: var(--btn-text-hover-bg, #f1f0f0); }
.bos-btn--text:active { background-color: var(--btn-text-pressed-bg, #e2e1e1); }

/* ── Disabled ───────────────────────────────────── */
.bos-btn:disabled,
.bos-btn--disabled {
  background-color: var(--btn-disabled-bg, #f1f0f0);
  color: var(--btn-disabled-text, #adaaaa);
  border-color: var(--btn-disabled-border, #e2e1e1);
  cursor: not-allowed;
  pointer-events: none;
}
/* Text disabled — no bg, just faded text */
.bos-btn--text:disabled,
.bos-btn--text.bos-btn--disabled {
  background-color: transparent;
  border-color: transparent;
}

/* ── Destructive ────────────────────────────────── */
/* Uses BOS Red color palette (not Turbo Pink) */
.bos-btn--destructive.bos-btn--fill {
  background-color: var(--btn-destructive-fill-bg, #e62822);
  color: var(--btn-destructive-fill-text, #fafafa);
}
.bos-btn--destructive.bos-btn--tonal {
  background-color: var(--btn-destructive-tonal-bg, #fce4e4);
  color: var(--btn-destructive-tonal-text, #e62822);
}
.bos-btn--destructive.bos-btn--outline-primary,
.bos-btn--destructive.bos-btn--outline-secondary {
  border-color: var(--btn-destructive-color, #e62822);
  color: var(--btn-destructive-color, #e62822);
}
.bos-btn--destructive.bos-btn--text {
  color: var(--btn-destructive-color, #e62822);
}

/* ── Icon-only ──────────────────────────────────── */
.bos-btn--icon {
  min-width: 0;
  width: 32px;          /* Default */
  padding: 0;
}
.bos-btn--icon.bos-btn--sm { width: 24px; }
.bos-btn--icon.bos-btn--lg { width: 40px; }
```

---

## HTML Usage

```html
<!-- Fill — Default -->
<button class="bos-btn bos-btn--fill">Button</button>

<!-- Fill — Small -->
<button class="bos-btn bos-btn--fill bos-btn--sm">Button</button>

<!-- Fill — Large -->
<button class="bos-btn bos-btn--fill bos-btn--lg">Button</button>

<!-- Tonal -->
<button class="bos-btn bos-btn--tonal">Button</button>

<!-- Outline Primary -->
<button class="bos-btn bos-btn--outline-primary">Button</button>

<!-- Outline Secondary -->
<button class="bos-btn bos-btn--outline-secondary">Button</button>

<!-- Text only -->
<button class="bos-btn bos-btn--text">Button</button>

<!-- Disabled -->
<button class="bos-btn bos-btn--fill" disabled>Button</button>

<!-- Destructive Fill -->
<button class="bos-btn bos-btn--fill bos-btn--destructive">Delete</button>

<!-- Icon-only Fill -->
<button class="bos-btn bos-btn--fill bos-btn--icon" aria-label="Action">
  <svg width="16" height="16">...</svg>
</button>

<!-- With left icon -->
<button class="bos-btn bos-btn--fill">
  <svg width="16" height="16">...</svg>
  Button
</button>
```

---

## Usage Notes

- Use **Fill** for the primary call-to-action on a page — limit to one per view.
- Use **Tonal** for secondary actions that need some visual weight.
- Use **Outline Primary** for important secondary actions alongside a Fill button.
- Use **Outline Secondary** for neutral or cancel actions.
- Use **Text** for low-priority or inline actions.
- **Destructive** variants are for irreversible actions (delete, remove). Always confirm with a dialog.
- Icon buttons must have `aria-label` for accessibility.
