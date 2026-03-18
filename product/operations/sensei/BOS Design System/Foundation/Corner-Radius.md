# Corner Radius

> **Page:** 🔘 Corner radius
> **Type:** Foundation — Design Tokens
> **Scale system:** Semantic tokens mapped to `#scale/*` primitives

---

## CSS Custom Properties

```css
:root {
  --corner-none: 0px;      /* #scale/none  — 0 rem    */
  --corner-2xs:  2px;      /* #scale/50    — 0.125rem */
  --corner-xs:   4px;      /* #scale/100   — 0.25rem  */
  --corner-s:    6px;      /* #scale/150   — 0.375rem */
  --corner-m:    8px;      /* #scale/200   — 0.5rem   */
  --corner-l:    16px;     /* #scale/400   — 1rem     */
  --corner-xl:   20px;     /* #scale/500   — 1.25rem  */
  --corner-2xl:  24px;     /* #scale/600   — 1.5rem   */
  --corner-full: 9999px;   /* #scale/full  — pill/circle */
}
```

---

## Token Table

| Token | Scale ref | px | rem | Visual |
|---|---|---|---|---|
| `none` | `#scale/none` | 0 px | 0 | Sharp square |
| `2xs` | `#scale/50` | 2 px | 0.125 | Almost square |
| `xs` | `#scale/100` | 4 px | 0.25 | Subtle round |
| `s` | `#scale/150` | 6 px | 0.375 | Slight round |
| `m` | `#scale/200` | 8 px | 0.5 | Default card/button |
| `l` | `#scale/400` | 16 px | 1 | Rounder card |
| `xl` | `#scale/500` | 20 px | 1.25 | Very round |
| `2xl` | `#scale/600` | 24 px | 1.5 | Highly round |
| `full` | `#scale/full` | 9999 px | 624.938 | Pill / Circle |

---

## Usage in CSS

```css
/* Buttons */
.bos-btn        { border-radius: var(--corner-m); }   /* 8px  */

/* Cards */
.bos-card       { border-radius: var(--corner-l); }   /* 16px */

/* Tags / Badges */
.bos-tag        { border-radius: var(--corner-full); } /* pill */

/* Input fields */
.bos-input      { border-radius: var(--corner-xs); }  /* 4px  */

/* Modal */
.bos-modal      { border-radius: var(--corner-m); }   /* 8px  */
```

---

## Usage Notes

- `full` (9999px) is the standard way to make a **pill shape** or **circle** — use it for tags, avatars, and status badges.
- Never mix `border-radius` raw values with token values in the same component.
