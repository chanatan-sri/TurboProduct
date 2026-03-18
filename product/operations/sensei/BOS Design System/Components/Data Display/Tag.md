# Tag

> **Page:** ↪︎ Tag
> **Type:** Component — Data Display

---

## Overview

Tags are small labeled pills for categorizing, filtering, or displaying status. BOS provides **Light** and **Dark** tone variants across multiple semantic colors, two sizes, and specialized status/DPD sets.

---

## Sizes

| Size | Height | Corner Radius |
|---|---|---|
| Default | 32px | `var(--corner/m, 8px)` |
| Small | 30px | `var(--corner/m, 8px)` |

---

## Tones

| Tone | Background | Text / Icon | Border |
|---|---|---|---|
| `Light` | Color/50 tint | Matching color | Color/100 |
| `Dark` | Color/400–500 | White `#ffffff` | Matching dark |

---

## Color Types — Light Tone

| Type | Background | Icon + Text Color | Stroke |
|---|---|---|---|
| `Default` (neutral) | `#f1f0f0` | `#242424` | `#e2e1e1` |
| `Blue` | `color/primary/50` ≈ `#e8f2fb` | `#2d79c8` | `color/primary/100` |
| `Green` | `color/success/50` ≈ `#e6f7ed` | `color/success/500☆` | `color/success/100` |
| `Yellow` | `color/caution/50` ≈ `#fffbe6` | `color/caution/500☆` | `color/caution/100` |
| `Orange` | `color/warning/50` ≈ `#fff4e6` | `color/warning/500☆` | `color/warning/100` |
| `Red` | `color/error/50` → `#fff2f0` | `#e62822` | `color/error/100` |
| `Purple` | `color/purple/50` | `color/purple/500☆` | `color/purple/100` |
| `Pink` | `color/pink/50` | `color/pink/500☆` | `color/pink/100` |
| `Teal` | `color/teal/50` | `color/teal/500☆` | `color/teal/100` |

## Color Types — Dark Tone

| Type | Background | Text |
|---|---|---|
| `Default` (neutral dark) | `color/neutral/500☆` ≈ `#7a7676` | `#ffffff` |
| `Blue dark` | `color/primary/400` ≈ `#4a8fd4` | `#ffffff` |
| `Green dark` | `color/success/300` | `#ffffff` |
| `Yellow dark` | `color/caution/200` | `#242424` |
| `Orange dark` | `color/warning/300` | `#ffffff` |
| `Salmon dark` | `color/error/400` ≈ `#ed4e4a` | `#ffffff` |
| `Red dark` | `color/error/400` ≈ `#ed4e4a` | `#ffffff` |
| `Black` | `#242424` | `#ffffff` |
| `Teal dark` | `color/teal/300` | `#ffffff` |

---

## Tag-Status (Workflow)

Pre-defined status tags for loan lifecycle states:

| Status | Description |
|---|---|
| Draft | Initial state |
| Pending Approve | Awaiting approval |
| Returned | Sent back for revision |
| Pending QA | Under quality review |
| Disburse | Disbursement in progress |
| Completed | Green — successfully finished |
| Reject | Red — rejected |
| Cancelled | Cancelled |
| Done | Completed/closed |

---

## Tag-DPD (Days Past Due)

Specialized tags for credit DPD statuses: `Current`, `30 DPD`, `60 DPD`, `90 DPD`, `XDPD`, each combined with legal case status (ไม่มี, ระหว่างฟ้องร้อง, เคยถูกฟ้องร้อง, บังคับคดี).

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `background-default` | `color/neutral/50` | `#fafafa` | Light default bg |
| `background-blue` | `color/primary/50` | `#e8f2fb` | Light blue bg |
| `background-green` | `color/success/50` | `#e6f7ed` | Light green bg |
| `background-yellow` | `color/caution/50` | `#fffbe6` | Light yellow bg |
| `background-orange` | `color/warning/50` | `#fff4e6` | Light orange bg |
| `background-red` | `color/error/50` | `#fff2f0` | Light red bg |
| `icon-default` | `color/Neutral/600` | `#625e5e` | Default icon |
| `icon-inverse` | `color/Neutral/0-white` | `#ffffff` | Dark tone icon |
| `text-default` | `color/Neutral/900-black` | `#242424` | Default text |
| `text-inverse` | `color/Neutral/0-white` | `#ffffff` | Dark tone text |
| `stroke-default` | `color/Neutral/100` | `#e2e1e1` | Default border |

---

## CSS Custom Properties

```css
:root {
  /* Light tone */
  --tag-bg-default:     #f1f0f0;
  --tag-bg-blue:        #e8f2fb;
  --tag-bg-green:       #e6f7ed;
  --tag-bg-yellow:      #fffbe6;
  --tag-bg-orange:      #fff4e6;
  --tag-bg-red:         #fff2f0;

  /* Dark tone */
  --tag-bg-blue-dark:   #4a8fd4;
  --tag-bg-green-dark:  #4db87d;
  --tag-bg-red-dark:    #ed4e4a;
  --tag-bg-black:       #242424;

  /* Text */
  --tag-text-default:   #242424;
  --tag-text-blue:      #2d79c8;
  --tag-text-green:     #00a848;
  --tag-text-red:       #e62822;
  --tag-text-inverse:   #ffffff;

  /* Stroke */
  --tag-stroke-default: #e2e1e1;
  --tag-stroke-blue:    #c5ddf5;
  --tag-stroke-green:   #a5dbb8;
  --tag-stroke-red:     #fcc4c0;
}
```

---

## CSS Classes

```css
/* ── Base tag ───────────────────────────────────── */
.bos-tag {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-xs, 4px);
  height: 32px;
  padding: var(--space-xs, 4px) var(--space-m, 8px);
  border-radius: var(--corner-m, 8px);
  border: 1px solid var(--tag-stroke-default, #e2e1e1);
  background-color: var(--tag-bg-default, #f1f0f0);
  font-family: var(--font-family);
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-regular);
  color: var(--tag-text-default, #242424);
  white-space: nowrap;
}

.bos-tag svg {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

/* ── Size ───────────────────────────────────────── */
.bos-tag--sm { height: 30px; }

/* ── Light color variants ───────────────────────── */
.bos-tag--blue {
  background-color: var(--tag-bg-blue, #e8f2fb);
  border-color: var(--tag-stroke-blue, #c5ddf5);
  color: var(--tag-text-blue, #2d79c8);
}

.bos-tag--green {
  background-color: var(--tag-bg-green, #e6f7ed);
  border-color: var(--tag-stroke-green, #a5dbb8);
  color: var(--tag-text-green, #00a848);
}

.bos-tag--yellow {
  background-color: var(--tag-bg-yellow, #fffbe6);
  border-color: #fef3b2;
  color: #a37800;
}

.bos-tag--orange {
  background-color: var(--tag-bg-orange, #fff4e6);
  border-color: #ffd9a8;
  color: #eb6b00;
}

.bos-tag--red {
  background-color: var(--tag-bg-red, #fff2f0);
  border-color: var(--tag-stroke-red, #fcc4c0);
  color: var(--tag-text-red, #e62822);
}

/* ── Dark color variants ────────────────────────── */
.bos-tag--dark {
  border-color: transparent;
  color: var(--tag-text-inverse, #ffffff);
}

.bos-tag--blue-dark  { background-color: var(--tag-bg-blue-dark, #4a8fd4); border-color: transparent; color: #ffffff; }
.bos-tag--green-dark { background-color: var(--tag-bg-green-dark, #4db87d); border-color: transparent; color: #ffffff; }
.bos-tag--red-dark   { background-color: var(--tag-bg-red-dark, #ed4e4a); border-color: transparent; color: #ffffff; }
.bos-tag--black      { background-color: var(--tag-bg-black, #242424); border-color: transparent; color: #ffffff; }

/* ── Close button ───────────────────────────────── */
.bos-tag__close {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  cursor: pointer;
  opacity: 0.6;
}

.bos-tag__close:hover { opacity: 1; }
```

---

## HTML Usage

```html
<!-- Light tone — default -->
<span class="bos-tag">
  <svg width="16" height="16"><!-- icon --></svg>
  Text
  <svg class="bos-tag__close" width="16" height="16"><!-- × --></svg>
</span>

<!-- Light tone — blue -->
<span class="bos-tag bos-tag--blue">Active</span>

<!-- Light tone — green — small -->
<span class="bos-tag bos-tag--green bos-tag--sm">Completed</span>

<!-- Light tone — red -->
<span class="bos-tag bos-tag--red">Rejected</span>

<!-- Light tone — orange -->
<span class="bos-tag bos-tag--orange">Warning</span>

<!-- Dark tone — blue -->
<span class="bos-tag bos-tag--blue-dark">Approved</span>

<!-- Dark tone — black -->
<span class="bos-tag bos-tag--black">Closed</span>
```

---

## Usage Notes

- Use **Light** tone for most UI contexts (light backgrounds).
- Use **Dark** tone for high-contrast or emphasis scenarios.
- Tags with a close (×) button are dismissible filter chips.
- **Tag-Status** and **Tag-DPD** are predefined application-specific tag sets — use those instances rather than building custom colors.
- Icon slot is optional — omit `<svg>` if not needed.
