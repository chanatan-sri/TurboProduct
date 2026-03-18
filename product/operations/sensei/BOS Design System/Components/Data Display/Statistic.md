# Statistic

> **Page:** ↪︎ Statistic
> **Type:** Component — Data Display

---

## Overview

Displays a key metric with an optional label, trend indicator, status badge, icon, and call-to-action button. Used in dashboards and summary cards.

---

## Anatomy

| Part | Description |
|---|---|
| Badge/Status | Optional status dot + label above the value |
| Value | Large numeric figure (22px) — main metric |
| Icon | Optional 24px icon left of the value |
| Trend | Small row below value — direction arrow + percentage |
| Button | Optional CTA button at the bottom |

---

## Trend Types

| Type | Direction | Color | Token |
|---|---|---|---|
| `Up` | ▲ arrow | Green | `statistic/text/positive` → `color/success/400` |
| `Flat` | → arrow | Neutral | `statistic/text/default` → `#242424` |
| `Down` | ▼ arrow | Red | `statistic/text/danger` → `color/error/500☆` |

---

## Design Tokens

| Token | BOS Color | Hex | Usage |
|---|---|---|---|
| `text-default` | `color/Neutral/900-black` | `#242424` | Value + icon + flat trend text |
| `text-subtle` | `color/neutral/600` | `#625e5e` | Label / title text |
| `text-positive` | `color/success/400` | `#00a848` | Up trend text + icon |
| `text-danger` | `color/error/500☆` | `#e62822` | Down trend text + icon |
| `icon-default` | `color/Neutral/900-black` | `#242424` | Icon next to value |

---

## Typography

| Element | Size | Line Height | Weight |
|---|---|---|---|
| Value digit | 22px (`scale/550`) | 40px (`scale/1000`) | Regular |
| Label / trend text | 12px (`scale/300`) | 22px (`scale/550`) | Regular |
| Button | 14px | 26px | Regular |

---

## CSS Custom Properties

```css
:root {
  --stat-text:         #242424;   /* color/Neutral/900-black */
  --stat-text-subtle:  #625e5e;   /* color/neutral/600 */
  --stat-positive:     #00a848;   /* color/success/400 */
  --stat-danger:       #e62822;   /* color/error/500☆ */
}
```

---

## CSS Classes

```css
/* ── Statistic block ─────────────────────────────── */
.bos-statistic {
  display: flex;
  flex-direction: column;
  gap: var(--space-m, 8px);
  align-items: flex-start;
  font-family: var(--font-family);
}

/* Inner group (status + value + trend) */
.bos-statistic__body {
  display: flex;
  flex-direction: column;
  gap: var(--space-xs, 4px);
  align-items: flex-start;
}

/* ── Label ───────────────────────────────────────── */
.bos-statistic__label {
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  font-weight: var(--font-weight-regular);
  color: var(--stat-text-subtle, #625e5e);
}

/* ── Value row ───────────────────────────────────── */
.bos-statistic__value {
  display: flex;
  align-items: center;
  gap: var(--space-xs, 4px);
}

.bos-statistic__value-icon {
  width: 24px;
  height: 24px;
  flex-shrink: 0;
  color: var(--stat-text, #242424);
}

.bos-statistic__digit {
  font-size: 22px;   /* scale/550 */
  line-height: 40px; /* scale/1000 */
  font-weight: var(--font-weight-regular);
  color: var(--stat-text, #242424);
  white-space: nowrap;
}

/* ── Trend ───────────────────────────────────────── */
.bos-statistic__trend {
  display: flex;
  align-items: center;
  gap: var(--space-xs, 4px);
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height);
  font-weight: var(--font-weight-regular);
  color: var(--stat-text-subtle, #625e5e);
}

.bos-statistic__trend-title {
  color: var(--stat-text-subtle, #625e5e);
}

.bos-statistic__trend-value {
  display: flex;
  align-items: center;
  gap: var(--space-xs, 4px);
}

.bos-statistic__trend-icon {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

.bos-statistic__trend-pct {
  white-space: nowrap;
}

/* Trend direction colors */
.bos-statistic--up   .bos-statistic__trend-value { color: var(--stat-positive, #00a848); }
.bos-statistic--down .bos-statistic__trend-value { color: var(--stat-danger, #e62822); }
.bos-statistic--flat .bos-statistic__trend-value { color: var(--stat-text, #242424); }
```

---

## HTML Usage

```html
<!-- Full statistic with trend up -->
<div class="bos-statistic bos-statistic--up">
  <div class="bos-statistic__body">
    <!-- Optional status badge -->
    <div class="bos-badge-status bos-badge-status--warning">
      <span class="bos-badge-status__dot"></span>
      Warning
    </div>

    <!-- Value -->
    <div class="bos-statistic__value">
      <svg class="bos-statistic__value-icon" width="24" height="24"><!-- icon --></svg>
      <span class="bos-statistic__digit">112,893</span>
    </div>

    <!-- Trend -->
    <div class="bos-statistic__trend">
      <span class="bos-statistic__trend-title">trend title</span>
      <div class="bos-statistic__trend-value">
        <svg class="bos-statistic__trend-icon" width="16" height="16"><!-- up arrow --></svg>
        <span class="bos-statistic__trend-pct">70.5%</span>
      </div>
    </div>
  </div>

  <!-- Optional CTA button -->
  <button class="bos-btn bos-btn--fill">Button</button>
</div>

<!-- Minimal — value only -->
<div class="bos-statistic">
  <div class="bos-statistic__body">
    <span class="bos-statistic__label">Total Applications</span>
    <div class="bos-statistic__value">
      <span class="bos-statistic__digit">4,521</span>
    </div>
  </div>
</div>
```

---

## Usage Notes

- Use on dashboard cards, KPI panels, and summary rows.
- Trend direction determines color: Up = green, Down = red, Flat = neutral.
- The button is optional — use only when a drill-down action is available.
- Badge/Status above the value provides semantic context (e.g. "Warning", "Active").
