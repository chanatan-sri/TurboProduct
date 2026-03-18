# Shadow

> **Page:** ⬛️ Shadow
> **Type:** Foundation — Design Tokens
> **Color:** All shadows use `rgba(0,0,0,*)` — black at varying opacity and spread

---

## CSS Custom Properties

```css
:root {
  --shadow-none: none;
  --shadow-xs:   0px 1px  2px  0px  rgba(0, 0, 0, 0.05);
  --shadow-s:    0px 3px  3px  -1px rgba(0, 0, 0, 0.05);
  --shadow-m:    0px 4px  6px  -2px rgba(0, 0, 0, 0.05);
  --shadow-l:    0px 8px  16px -4px rgba(0, 0, 0, 0.05);
  --shadow-xl:   0px 12px 20px -8px rgba(0, 0, 0, 0.20);
}
```

---

## Token Table

| Token | CSS `box-shadow` value | Visual weight |
|---|---|---|
| `none` | `none` | No shadow |
| `xs` | `0px 1px 2px 0px rgba(0,0,0,0.05)` | Barely visible — subtle lift |
| `s` | `0px 3px 3px -1px rgba(0,0,0,0.05)` | Light card elevation |
| `m` | `0px 4px 6px -2px rgba(0,0,0,0.05)` | Default card / panel |
| `l` | `0px 8px 16px -4px rgba(0,0,0,0.05)` | Dropdown / popover |
| `xl` | `0px 12px 20px -8px rgba(0,0,0,0.20)` | Modal / dialog |

---

## Usage in CSS

```css
/* Buttons / small chips */
.bos-chip        { box-shadow: var(--shadow-xs); }

/* Default card */
.bos-card        { box-shadow: var(--shadow-s); }

/* Raised card / table */
.bos-panel       { box-shadow: var(--shadow-m); }

/* Dropdown / tooltip */
.bos-dropdown    { box-shadow: var(--shadow-l); }

/* Modal / dialog overlay */
.bos-modal       { box-shadow: var(--shadow-xl); }

/* Flat — no elevation */
.bos-flat        { box-shadow: var(--shadow-none); }
```

---

## Usage Notes

- `xl` uses **20% opacity** — significantly darker, reserved for modals and dialogs to reinforce the top-level overlay feeling.
- `xs`–`l` all use **5% opacity** — very subtle, providing just enough depth without visual noise.
- Use `none` explicitly on flat elements (e.g. inline tags, table rows) so intent is clear.
- Shadows work best on `#fafafa` or white backgrounds. Avoid using shadows on dark backgrounds.
