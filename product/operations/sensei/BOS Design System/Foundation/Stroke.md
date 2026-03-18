# Stroke

> **Page:** 🔳 Stroke
> **Type:** Foundation — Design Tokens
> **Scale system:** Semantic tokens mapped to `#scale/*` primitives

---

## CSS Custom Properties

```css
:root {
  --stroke-none: 0px;   /* #scale/none — 0 rem    */
  --stroke-xs:   1px;   /* #scale/25   — 0.063rem */
  --stroke-s:    2px;   /* #scale/50   — 0.125rem */
  --stroke-m:    3px;   /* #scale/75   — 0.188rem */
  --stroke-l:    4px;   /* #scale/100  — 0.25rem  */
}
```

---

## Token Table

| Token | Scale ref | px | rem |
|---|---|---|---|
| `none` | `#scale/none` | 0 px | 0 |
| `xs` | `#scale/25` | 1 px | 0.063 |
| `s` | `#scale/50` | 2 px | 0.125 |
| `m` | `#scale/75` | 3 px | 0.188 |
| `l` | `#scale/100` | 4 px | 0.25 |

---

## Usage in CSS

```css
/* Default border (most common) */
.bos-card   { border: var(--stroke-xs) solid #e2e1e1; }  /* 1px */

/* Input focus ring */
.bos-input:focus { border-width: var(--stroke-s); }       /* 2px */

/* Emphasis border */
.bos-highlight { border-width: var(--stroke-m); }         /* 3px */

/* Heavy border / divider */
.bos-divider   { border-width: var(--stroke-l); }         /* 4px */
```

---

## Usage Notes

- `xs` (1px) is the default for cards, inputs, modals, and tables.
- `s` (2px) is used for **focus states** to indicate active interaction.
- Use `none` explicitly when removing a border so intent is clear.
