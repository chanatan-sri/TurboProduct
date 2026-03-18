# Upload

> **Page:** ↪︎ Upload
> **Type:** Component — Data Entry

---

## Overview

A file upload component with two display variants: a **Upload Zone** (drag-and-drop area) and a **Picture Card** (thumbnail grid). The upload zone supports drag-and-drop or click-to-browse. Picture cards show file thumbnails and support multiple sizes, upload progress, and error states.

---

## Variants

| Variant | Description |
|---|---|
| `Upload Zone` | Full-width dashed drop zone for any file type |
| `Picture Card` | Square thumbnail card for image uploads |

---

## Upload Zone

### Dimensions

| Property | Value |
|---|---|
| Width | 100% (full width) |
| Corner radius | 6px |
| Border | 1px dashed `#c7c5c5` |
| Background | `#f1f0f0` |
| Padding | 24px |

### Content

| Element | Style |
|---|---|
| Upload icon | 24px, color `#adaaaa` |
| Title | 14px/26px semibold, `#242424` |
| Subtitle | 12px/22px regular, `#625e5e` |

### Zone States

| State | Border | Background |
|---|---|---|
| `Default` | 1px dashed `#c7c5c5` | `#f1f0f0` |
| `Hover / Drag over` | 1px dashed `#2d79c8` | `#e8f2fb` |
| `Disabled` | 1px dashed `#c7c5c5` | `#f1f0f0`, opacity 50% |

---

## Picture Card

### Sizes

| Size | Dimensions |
|---|---|
| `Small` | 120 × 142px |
| `Large` | 250 × 272px |

### Card Dimensions

| Property | Value |
|---|---|
| Card (item) size | 104 × 104px |
| Corner radius | 2px |
| Border | 1px solid `#d9d9d9` |
| Thumbnail bg | `#fafafa` |

### Card States

| State | Description |
|---|---|
| `Default` | File ready — shows thumbnail |
| `Hover` | Overlay with actions (preview, delete) |
| `Uploading` | Shows progress bar or spinner |
| `Error` | Red border and error icon |

---

## Design Tokens

| Token | Hex | Usage |
|---|---|---|
| `color/neutral/100` | `#f1f0f0` | Upload zone bg |
| `color/neutral/250` | `#c7c5c5` | Zone border (default) |
| `color/primary/500☆` | `#2d79c8` | Zone border (drag over) |
| `color/primary/50` | `#e8f2fb` | Zone bg (drag over) |
| `color/neutral/300` | `#adaaaa` | Upload icon |
| `color/neutral/900-black` | `#242424` | Zone title |
| `color/neutral/600` | `#625e5e` | Zone subtitle |
| `color/neutral/200` | `#d9d9d9` | Card border |
| `color/error/500☆` | `#e62822` | Error border |

---

## CSS Custom Properties

```css
:root {
  --upload-zone-bg:         #f1f0f0;
  --upload-zone-border:     #c7c5c5;
  --upload-zone-border-over:#2d79c8;
  --upload-zone-bg-over:    #e8f2fb;
  --upload-icon:            #adaaaa;
  --upload-title:           #242424;
  --upload-subtitle:        #625e5e;
  --upload-card-border:     #d9d9d9;
  --upload-card-border-error:#e62822;
}
```

---

## CSS Classes

```css
/* ── Upload Zone ──────────────────────────── */
.bos-upload-zone {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 4px;
  width: 100%;
  padding: 24px;
  background-color: var(--upload-zone-bg, #f1f0f0);
  border: 1px dashed var(--upload-zone-border, #c7c5c5);
  border-radius: var(--corner-s, 6px);
  text-align: center;
  cursor: pointer;
  font-family: var(--font-family);
}

.bos-upload-zone:hover,
.bos-upload-zone--dragover {
  background-color: var(--upload-zone-bg-over, #e8f2fb);
  border-color: var(--upload-zone-border-over, #2d79c8);
}

.bos-upload-zone--disabled {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

.bos-upload-zone__icon {
  width: 24px;
  height: 24px;
  color: var(--upload-icon, #adaaaa);
  flex-shrink: 0;
}

.bos-upload-zone__title {
  font-size: var(--text-body-size);         /* 14px */
  line-height: var(--text-body-line-height); /* 26px */
  font-weight: var(--font-weight-semibold, 600);
  color: var(--upload-title, #242424);
}

.bos-upload-zone__subtitle {
  font-size: var(--text-small-size);         /* 12px */
  line-height: var(--text-small-line-height); /* 22px */
  font-weight: var(--font-weight-regular);
  color: var(--upload-subtitle, #625e5e);
}

/* ── Picture Card List ────────────────────── */
.bos-upload-cards {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  font-family: var(--font-family);
}

/* ── Picture Card Item ────────────────────── */
.bos-upload-card {
  position: relative;
  width: 104px;
  height: 104px;
  border-radius: var(--corner-xs, 2px);
  border: 1px solid var(--upload-card-border, #d9d9d9);
  background-color: #fafafa;
  overflow: hidden;
  cursor: pointer;
  flex-shrink: 0;
}

.bos-upload-card--sm {
  width: 120px;
  height: 142px;
}

.bos-upload-card--lg {
  width: 250px;
  height: 272px;
}

.bos-upload-card__thumbnail {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Hover overlay */
.bos-upload-card__overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  opacity: 0;
  transition: opacity 0.15s;
}

.bos-upload-card:hover .bos-upload-card__overlay {
  opacity: 1;
}

.bos-upload-card__overlay-btn {
  width: 24px;
  height: 24px;
  color: #ffffff;
  cursor: pointer;
}

/* Uploading state */
.bos-upload-card--uploading .bos-upload-card__overlay {
  opacity: 1;
  background: rgba(255, 255, 255, 0.7);
  flex-direction: column;
}

/* Error state */
.bos-upload-card--error {
  border-color: var(--upload-card-border-error, #e62822);
}

/* Add card button */
.bos-upload-card--add {
  display: flex;
  align-items: center;
  justify-content: center;
  border-style: dashed;
  cursor: pointer;
  color: var(--upload-icon, #adaaaa);
}

.bos-upload-card--add svg {
  width: 20px;
  height: 20px;
}
```

---

## HTML Usage

```html
<!-- Upload Zone -->
<div class="bos-upload-zone">
  <svg class="bos-upload-zone__icon" width="24" height="24"><!-- upload icon --></svg>
  <p class="bos-upload-zone__title">ลากไฟล์มาวางที่นี่ หรือคลิกเพื่ออัปโหลด</p>
  <p class="bos-upload-zone__subtitle">ขนาดแต่ละไฟล์ไม่เกิน 25 MB รองรับ PDF, JPG, PNG</p>
</div>

<!-- Upload Zone — drag over -->
<div class="bos-upload-zone bos-upload-zone--dragover">
  <svg class="bos-upload-zone__icon" width="24" height="24"><!-- upload icon --></svg>
  <p class="bos-upload-zone__title">วางไฟล์ที่นี่</p>
</div>

<!-- Picture Card — default (with thumbnail) -->
<div class="bos-upload-cards">
  <div class="bos-upload-card">
    <img class="bos-upload-card__thumbnail" src="file-thumbnail.jpg" alt="ไฟล์ที่อัปโหลด" />
    <div class="bos-upload-card__overlay">
      <svg class="bos-upload-card__overlay-btn" width="24" height="24"><!-- eye icon --></svg>
      <svg class="bos-upload-card__overlay-btn" width="24" height="24"><!-- trash icon --></svg>
    </div>
  </div>

  <!-- Uploading -->
  <div class="bos-upload-card bos-upload-card--uploading">
    <div class="bos-upload-card__overlay">
      <!-- spinner or progress -->
      <svg width="24" height="24"><!-- spinner --></svg>
      <span style="font-size: 12px; color: #625e5e;">กำลังอัปโหลด...</span>
    </div>
  </div>

  <!-- Error -->
  <div class="bos-upload-card bos-upload-card--error">
    <img class="bos-upload-card__thumbnail" src="file-thumbnail.jpg" alt="" />
    <div class="bos-upload-card__overlay" style="opacity:1; background: rgba(230,40,34,0.15);">
      <svg class="bos-upload-card__overlay-btn" width="24" height="24" style="color:#e62822;"><!-- error icon --></svg>
    </div>
  </div>

  <!-- Add button -->
  <div class="bos-upload-card bos-upload-card--add">
    <svg width="20" height="20"><!-- plus icon --></svg>
  </div>
</div>

<!-- Picture Card — Large size -->
<div class="bos-upload-cards">
  <div class="bos-upload-card bos-upload-card--lg">
    <img class="bos-upload-card__thumbnail" src="file-thumbnail.jpg" alt="" />
    <div class="bos-upload-card__overlay">
      <svg class="bos-upload-card__overlay-btn" width="24" height="24"><!-- eye icon --></svg>
      <svg class="bos-upload-card__overlay-btn" width="24" height="24"><!-- trash icon --></svg>
    </div>
  </div>
</div>
```

---

## Usage Notes

- The upload zone accepts click (opens file browser) and drag-and-drop. Add `bos-upload-zone--dragover` on `dragenter`/`dragover` events; remove on `dragleave`/`drop`.
- Picture card default size is 104×104px; use `--sm` (120×142px) or `--lg` (250×272px) for size variants.
- Picture card corner radius is 2px — intentionally smaller than other components.
- Always include an "add" card at the end of a picture card list to allow uploading more files.
- For the uploading state, show a spinner or progress indicator inside the card overlay.
- Maximum file size guidance (25 MB) is displayed in the zone subtitle — update as needed.
