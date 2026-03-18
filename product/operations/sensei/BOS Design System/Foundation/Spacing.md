# Spacing

> **Page:** 📏 Spacing
> **Type:** Foundation — Design Tokens
> **Scale system:** Semantic tokens mapped to `#scale/*` primitives

---

## CSS Custom Properties

```css
:root {
  --space-none: 0px;      /* #scale/none  — 0 rem   */
  --space-2xs:  2px;      /* #scale/50    — 0.125rem */
  --space-xs:   4px;      /* #scale/100   — 0.25rem  */
  --space-s:    6px;      /* #scale/150   — 0.375rem */
  --space-m:    8px;      /* #scale/200   — 0.5rem   */
  --space-l:    12px;     /* #scale/300   — 0.75rem  */
  --space-xl:   16px;     /* #scale/400   — 1rem     */
  --space-2xl:  20px;     /* #scale/500   — 1.25rem  */
  --space-3xl:  24px;     /* #scale/600   — 1.5rem   */
}
```

---

## Token Table

| Token | Scale ref | px | rem |
|---|---|---|---|
| `none` | `#scale/none` | 0 px | 0 |
| `2xs` | `#scale/50` | 2 px | 0.125 |
| `xs` | `#scale/100` | 4 px | 0.25 |
| `s` | `#scale/150` | 6 px | 0.375 |
| `m` | `#scale/200` | 8 px | 0.5 |
| `l` | `#scale/300` | 12 px | 0.75 |
| `xl` | `#scale/400` | 16 px | 1 |
| `2xl` | `#scale/500` | 20 px | 1.25 |
| `3xl` | `#scale/600` | 24 px | 1.5 |

---

## Usage in CSS

```css
/* Padding */
.card { padding: var(--space-l); }          /* 12px */

/* Gap */
.row  { gap: var(--space-m); }              /* 8px  */

/* Margin */
.section { margin-bottom: var(--space-xl); } /* 16px */
```

---

## Usage Notes

- Base font-size assumed **16px** for rem conversions.
- Use semantic token names (`--space-m`) in code — never hardcode px values.
- `none` is explicit zero; use it instead of `0` so intent is clear.
