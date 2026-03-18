# Typography

> **Page:** 🔤 Typography
> **Type:** Foundation — Design Tokens
> **Active font:** Sarabun only

---

## Typeface

| Token | Font family |
|---|---|
| `font/typeface/sarabun` | Sarabun |

```css
@import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600&display=swap');
```

---

## Font Weight

| Token | Value | CSS keyword |
|---|---|---|
| `font/weight/300` | 300 | Light |
| `font/weight/400` | 400 | Regular |
| `font/weight/500` | 500 | Medium |
| `font/weight/600` | 600 | Semibold |

```css
:root {
  --font-weight-light:    300;
  --font-weight-regular:  400;
  --font-weight-medium:   500;
  --font-weight-semibold: 600;
}
```

---

## Text Style Set

| Set | Style | Size | Rem | Line Height | Weight |
|---|---|---|---|---|---|
| **Heading M** | Semibold | 22 px | 1.375 | 40 px | 600 |
| **Heading M** | Regular | 22 px | 1.375 | 40 px | 400 |
| **Heading S** | Semibold | 18 px | 1.125 | 32 px | 600 |
| **Heading S** | Regular | 18 px | 1.125 | 32 px | 400 |
| **Sub-heading M** | Semibold | 16 px | 1 | 28 px | 600 |
| **Sub-heading M** | Regular | 16 px | 1 | 28 px | 400 |
| **Body** | Semibold | 14 px | 0.875 | 26 px | 600 |
| **Body** | Regular | 14 px | 0.875 | 26 px | 400 |
| **Body** | Underline | 14 px | 0.875 | 26 px | 400 |
| **Small text** | Regular | 12 px | 0.75 | 22 px | 400 |

---

## CSS Custom Properties

```css
:root {
  --font-family: 'Sarabun', sans-serif;

  /* Heading M */
  --text-heading-m-size:        22px;
  --text-heading-m-line-height: 40px;

  /* Heading S */
  --text-heading-s-size:        18px;
  --text-heading-s-line-height: 32px;

  /* Sub-heading M */
  --text-subheading-m-size:        16px;
  --text-subheading-m-line-height: 28px;

  /* Body */
  --text-body-size:        14px;
  --text-body-line-height: 26px;

  /* Small text */
  --text-small-size:        12px;
  --text-small-line-height: 22px;
}
```

---

## CSS Classes

```css
/* ── Base ──────────────────────────────────────── */
body {
  font-family: var(--font-family);
  font-size: var(--text-body-size);
  font-weight: var(--font-weight-regular);
  line-height: var(--text-body-line-height);
  color: #242424;
}

/* ── Heading M ─────────────────────────────────── */
.bos-heading-m {
  font-size: var(--text-heading-m-size);       /* 22px */
  line-height: var(--text-heading-m-line-height); /* 40px */
}
.bos-heading-m--semibold { font-weight: var(--font-weight-semibold); }
.bos-heading-m--regular  { font-weight: var(--font-weight-regular); }

/* ── Heading S ─────────────────────────────────── */
.bos-heading-s {
  font-size: var(--text-heading-s-size);       /* 18px */
  line-height: var(--text-heading-s-line-height); /* 32px */
}
.bos-heading-s--semibold { font-weight: var(--font-weight-semibold); }
.bos-heading-s--regular  { font-weight: var(--font-weight-regular); }

/* ── Sub-heading M ─────────────────────────────── */
.bos-subheading-m {
  font-size: var(--text-subheading-m-size);       /* 16px */
  line-height: var(--text-subheading-m-line-height); /* 28px */
}
.bos-subheading-m--semibold { font-weight: var(--font-weight-semibold); }
.bos-subheading-m--regular  { font-weight: var(--font-weight-regular); }

/* ── Body ──────────────────────────────────────── */
.bos-body {
  font-size: var(--text-body-size);       /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
}
.bos-body--semibold { font-weight: var(--font-weight-semibold); }
.bos-body--regular  { font-weight: var(--font-weight-regular); }
.bos-body--underline {
  font-weight: var(--font-weight-regular);
  text-decoration: underline;
}

/* ── Small text ────────────────────────────────── */
.bos-small {
  font-size: var(--text-small-size);       /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  font-weight: var(--font-weight-regular);
}
```

---

## HTML Usage Examples

```html
<!-- Heading M Semibold -->
<h2 class="bos-heading-m bos-heading-m--semibold">หัวข้อหลัก</h2>

<!-- Heading S Regular -->
<h3 class="bos-heading-s bos-heading-s--regular">หัวข้อรอง</h3>

<!-- Sub-heading -->
<h4 class="bos-subheading-m bos-subheading-m--semibold">หัวข้อย่อย</h4>

<!-- Body text -->
<p class="bos-body bos-body--regular">เนื้อหาปกติ</p>
<p class="bos-body bos-body--semibold">เนื้อหาตัวหนา</p>
<p class="bos-body bos-body--underline">เนื้อหาขีดเส้นใต้</p>

<!-- Small text -->
<span class="bos-small">ข้อความขนาดเล็ก</span>
```

---

## Usage Notes

- Use **Semibold (600)** for labels, column headers, button text, and emphasis.
- Use **Regular (400)** for body content, descriptions, and secondary text.
- Use **Underline** for inline links within body text only — not for standalone buttons.
- **Small text** (12px) is for captions, helper text, and table footnotes.
