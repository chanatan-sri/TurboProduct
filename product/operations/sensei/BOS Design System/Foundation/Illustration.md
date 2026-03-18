# Illustration

> **Page:** 🌠 Illustration
> **Type:** Foundation — Illustration & Image Library

---

## System Illustrations

Used for empty states, errors, and special UI moments.

| Token name | Display name | When to use |
|---|---|---|
| `illus/empty-color` | Empty — Default | Default empty state (coloured version) |
| `illus/empty-gray` | Empty — Gray | Subtle empty state (greyscale version) |
| `illus/empty-error` | Empty — Error | Error empty state |
| `illus/empty-search` | Empty — Search | No search results found |
| `illus/dip chip` | Dip chip | Custom branded moment |
| `illus/select file` | Select file | File upload / selection prompt |

---

## Image Component Sizes

Used for profile photos, thumbnails, and content images.

| Size token | px |
|---|---|
| `Size=XS` | 20 × 20 px |
| `Size=S` | 40 × 40 px |
| `Size=M` | 56 × 56 px |
| `Size=L` | 80 × 80 px |
| `Size=XL` | 120 × 120 px |
| `Size=XXL` | 160 × 160 px |

---

## Usage in HTML

```html
<!-- System illustration (empty state) -->
<div class="bos-empty-state">
  <img src="illustrations/empty-color.svg" alt="No data" class="bos-illus" />
  <p>ไม่พบข้อมูล</p>
</div>

<!-- Empty search result -->
<div class="bos-empty-state">
  <img src="illustrations/empty-search.svg" alt="No results" class="bos-illus" />
  <p>ไม่พบผลการค้นหา</p>
</div>

<!-- Profile image - size M -->
<img src="photo.jpg" alt="Profile" class="bos-image bos-image--m" />
```

```css
/* Illustration */
.bos-illus {
  display: block;
  margin: 0 auto;
}

/* Image sizes */
.bos-image       { border-radius: 50%; object-fit: cover; }
.bos-image--xs   { width: 20px;  height: 20px;  }
.bos-image--s    { width: 40px;  height: 40px;  }
.bos-image--m    { width: 56px;  height: 56px;  }
.bos-image--l    { width: 80px;  height: 80px;  }
.bos-image--xl   { width: 120px; height: 120px; }
.bos-image--xxl  { width: 160px; height: 160px; }
```

---

## Usage Notes

- Use `illus/empty-color` as the default empty state.
- Use `illus/empty-gray` when the empty state appears inside a subtle or secondary panel.
- Use `illus/empty-error` specifically for failed loads or system errors, not empty data.
- Illustration SVGs should be exported from Figma and stored in `assets/illustrations/`.
