
# Webready Images Workflow (`v2`)

Reusable GitHub Actions workflow for automatically generating optimized derived image assets from source images.

This workflow creates in a **single run**:

- standard webready JPEGs
- optional smaller JPEG variants
- optional WEBP variants

with automatic commit & push back to the caller repository.

---

## Repository

`JannisElef/webready-images-workflow`

Workflow file:

`/.github/workflows/webready-images.yml`

Recommended release tag:

`v2`

---

## Output Structure

Source images stay in:

```txt
assets/img/
```

Generated files are placed into:

```txt
assets/img/web/
assets/img/web/smaller/
assets/img/web/webp/
```

### Example

Source:

```txt
assets/img/catan/board.png
```

Generated:

```txt
assets/img/web/catan/board-web.jpg
assets/img/web/smaller/catan/board-smaller.jpg
assets/img/web/webp/catan/board-webp.webp
```

---

## Features

- single reusable workflow
- one checkout only
- one sharp install only
- one commit/push only
- no Git push race conditions
- preserves folder structure recursively
- skips unchanged files
- supports:
  - jpg/png/jpeg/webp/avif/tif/tiff/gif input
- auto rotate based on EXIF
- white flatten for transparent images
- mozjpeg optimization
- optional webp generation

---

## Caller Repository Usage

Create:

```txt
.github/workflows/webready-images.yml
```

with:

```yaml
name: webready images

on:
  workflow_dispatch:
  push:
    paths:
      - "assets/img/**"
      - "!assets/img/web/**"

permissions:
  contents: write

jobs:
  webready:
    uses: JannisElef/webready-images-workflow/.github/workflows/webready-images.yml@v2
    with:
      source-dir: assets/img

      output-dir: assets/img/web
      quality: 80
      suffix: -web
      max-width: 1024
      max-height: 1024

      generate-smaller: true
      smaller-quality: 65
      smaller-suffix: -smaller
      smaller-max-width: 512
      smaller-max-height: 512

      generate-webp: true
      webp-quality: 78
      webp-suffix: -webp
      webp-max-width: 1024
      webp-max-height: 1024
```

---

## Important Trigger Rule

The generated output directory must be excluded from the push trigger:

```yaml
- "!assets/img/web/**"
```

Otherwise generated commits would trigger the workflow again and create an infinite loop.

---

## Available Inputs

| Input | Type | Default | Description |
|------|------|---------|-------------|
| `source-dir` | string | `assets/img` | source image folder |
| `output-dir` | string | `assets/img/web` | root generated output folder |
| `quality` | number | `80` | jpeg quality for standard web images |
| `suffix` | string | `-web` | filename suffix for standard images |
| `max-width` | number | — | optional max width |
| `max-height` | number | — | optional max height |
| `generate-smaller` | boolean | `false` | enable smaller jpg generation |
| `smaller-quality` | number | `65` | jpeg quality for smaller images |
| `smaller-suffix` | string | `-smaller` | filename suffix |
| `smaller-max-width` | number | `512` | max width |
| `smaller-max-height` | number | `512` | max height |
| `generate-webp` | boolean | `false` | enable webp generation |
| `webp-quality` | number | `78` | webp quality |
| `webp-suffix` | string | `-webp` | filename suffix |
| `webp-max-width` | number | — | optional max width |
| `webp-max-height` | number | — | optional max height |

---

## Recommended Repository Convention

Keep original editable source images only in:

```txt
assets/img/
```

Never manually place files inside:

```txt
assets/img/web/
```

because everything inside `/web` is treated as generated derived output.

---

## Recommended HTML Usage Pattern

Standard responsive usage:

```html
<picture>
  <source srcset="/assets/img/web/webp/catan/board-webp.webp" type="image/webp">
  <source srcset="/assets/img/web/smaller/catan/board-smaller.jpg" media="(max-width: 768px)">
  <img src="/assets/img/web/catan/board-web.jpg" alt="Board">
</picture>
```

---

## Notes

Current workflow behavior:

- generates missing files
- regenerates changed source files

---
