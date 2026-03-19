# Chip

> **Page:** ↪︎ Chip
> **Type:** Component — Data Entry

---

## Overview

Chips are interactive pill-shaped controls for filtering, selecting, or toggling single options. They come in two types (Filled / Outlined), three sizes, and two selection states. Selected Filled chips use a bold primary fill; selected Outlined chips use a primary border.

---

## Types

| Type | Description |
|---|---|
| `Filled` | Solid background — light blue when unselected, primary blue when selected |
| `Outlined` | Border only — gray border when unselected, primary border when selected |

---

## Sizes

| Size | Height | Min-width |
|---|---|---|
| `Small` | 28px | 72px |
| `Default` | 32px | 72px |
| `Large` | 40px | 72px |

---

## States

| State | Description |
|---|---|
| `Default` | Resting |
| `Hover` | Mouse over |
| `Press` | Active press |
| `Disabled` | Not interactive — reduced opacity |

---

## Colors by Type & Selection

### Filled

| Selection | Background | Text |
|---|---|---|
| Unselected | `chip/background/filled-selected-default` = `#d2e4f8` | `#025fac` |
| Selected | `color/primary/500☆` = `#2d79c8` | `#fcfcfc` + check icon |

### Outlined

| Selection | Background | Border | Text |
|---|---|---|---|
| Unselected | `#fafafa` | `#adaaaa` | `#4a4646` |
| Selected | `#fafafa` | `#2d79c8` | `#025fac` |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `chip/background/filled-selected-default` | `#d2e4f8` | Filled unselected bg |
| `chip/text/filled-selected-default` | `#025fac` | Filled unselected text |
| `chip/background/outlined-deselect-default` | `#fafafa` | Outlined unselected bg |
| `chip/stroke/outlined-deselect-default` | `#adaaaa` | Outlined unselected border |
| `chip/text/outlined-deselect-default` | `#4a4646` | Outlined unselected text |

---

## CSS Custom Properties

```css
:root {
  --chip-bg-filled:          #d2e4f8;
  --chip-text-filled:        #025fac;
  --chip-bg-filled-selected: #2d79c8;
  --chip-text-selected:      #fcfcfc;
  --chip-bg-outlined:        #fafafa;
  --chip-stroke-outlined:    #adaaaa;
  --chip-text-outlined:      #4a4646;
  --chip-stroke-selected:    #2d79c8;
  --chip-text-outlined-selected: #025fac;
}
```

---

## CSS Classes

```css
/* ── Base chip ───────────────────────────────── */
.bos-chip {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-xs, 4px);
  height: 32px;
  min-width: 72px;
  padding: 0 var(--space-m, 8px);
  border-radius: var(--corner-full, 9999px);
  font-family: var(--font-family);
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  cursor: pointer;
  white-space: nowrap;
  border: 1px solid transparent;
}

/* ── Sizes ───────────────────────────────────── */
.bos-chip--sm  { height: 28px; }
.bos-chip--lg  { height: 40px; }

/* ── Filled ──────────────────────────────────── */
.bos-chip--filled {
  background-color: var(--chip-bg-filled, #d2e4f8);
  color: var(--chip-text-filled, #025fac);
}

.bos-chip--filled.bos-chip--selected {
  background-color: var(--chip-bg-filled-selected, #2d79c8);
  color: var(--chip-text-selected, #fcfcfc);
}

/* ── Outlined ────────────────────────────────── */
.bos-chip--outlined {
  background-color: var(--chip-bg-outlined, #fafafa);
  border-color: var(--chip-stroke-outlined, #adaaaa);
  color: var(--chip-text-outlined, #4a4646);
}

.bos-chip--outlined.bos-chip--selected {
  border-color: var(--chip-stroke-selected, #2d79c8);
  color: var(--chip-text-outlined-selected, #025fac);
}

/* ── Disabled ────────────────────────────────── */
.bos-chip--disabled {
  opacity: 0.4;
  cursor: not-allowed;
  pointer-events: none;
}

/* ── Check icon (selected filled) ───────────── */
.bos-chip__check {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}
```

---

## HTML Usage

```html
<!-- Filled — unselected -->
<button class="bos-chip bos-chip--filled">ทั้งหมด</button>

<!-- Filled — selected (with check icon) -->
<button class="bos-chip bos-chip--filled bos-chip--selected">
  <svg class="bos-chip__check" width="20" height="20"><!-- check --></svg>
  อนุมัติแล้ว
</button>

<!-- Outlined — unselected -->
<button class="bos-chip bos-chip--outlined">ร่าง</button>

<!-- Outlined — selected -->
<button class="bos-chip bos-chip--outlined bos-chip--selected">สำเร็จ</button>

<!-- Small — filled -->
<button class="bos-chip bos-chip--filled bos-chip--sm">ตัวกรอง</button>

<!-- Large — outlined -->
<button class="bos-chip bos-chip--outlined bos-chip--lg">ประเภท</button>

<!-- Disabled -->
<button class="bos-chip bos-chip--filled bos-chip--disabled" disabled>ไม่พร้อมใช้งาน</button>
```

---

## Usage Notes

- Use **Filled** chips for primary filter/toggle actions.
- Use **Outlined** chips for secondary or less prominent selections.
- Chips are pill-shaped (`border-radius: 9999px`) — do not use corner-m.
- Selected Filled chips show a check icon before the label.
- Chip groups should be laid out in a horizontal flex row with a gap.
