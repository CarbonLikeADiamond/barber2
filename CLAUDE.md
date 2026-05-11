# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static single-page website for Notorious Barbershop Kyjov. No build step, no framework, no package manager — the entire site is one `index.html` file with all CSS and JS inlined. Deploy target is Vercel (root dir: `.`).

## Development

```bash
python -m http.server 8080
# or
npx serve .
```

No tests, no linting, no CI.

## Image workflow

All new images must be converted to WebP before committing. Use Python + Pillow:

```python
from PIL import Image

# Portrait (team photos): 440×440, quality 85
img.crop((x, y, x+size, y+size)).resize((440, 440), Image.LANCZOS).save('out.webp', 'WEBP', quality=85, method=6)

# Landscape (hero): 1440×960 full + 750×500 mobile variant
# Gallery portraits: 900×1200 — Gallery landscapes: 1200×800
```

**Hero image** has two sizes for responsive loading:
- `images/hero.webp` — 1440×960 (desktop)
- `images/hero-sm.webp` — 750×500 (mobile, loaded on screens ≤750 w)

The `<link rel="preload">` in `<head>` uses `imagesrcset`/`imagesizes` to match the `<picture>` srcset — both must be kept in sync.

**Czech typography** (non-breaking spaces after single-letter prepositions) is pre-baked into the HTML statically. When editing text content, run this Python snippet to re-apply before committing — do NOT add a JS function for it (causes forced reflow):

```python
import re
PATTERN = re.compile(r'\b([aikovszuAIKOVSZU]) +')
# Apply only to text nodes (skip <script>/<style>) using a tag-aware state machine
```

## Architecture

Everything lives in `index.html` — CSS → JS → HTML in that order within the file.

**CSS custom properties** (`:root`): `--bg-*`, `--color-*`, `--font-*`, `--border-*`. Always use variables; never hardcode colours or fonts.

**Section order** (DOM): `<header #site-header>` → `<main>` → `#hero` → `#about` → `#services` → `#voucher` → `#team` → `#gallery` → `#contact` → `</main>` → `<footer #footer>`

**Shape dividers**: each section uses a CSS `::after` pseudo-element with a diagonal clip-path to visually cut into the next section's background colour. The mapping is in the `SHAPE DIVIDERS` CSS block — update it if section background colours change.

**Gallery**: CSS `columns: 4` masonry (no JS, no Grid). Items use `break-inside: avoid` and display at natural aspect ratio (`height: auto`). Responsive: 3 col @ 1024 px, 2 col @ 768 px, 1 col @ 600 px.

**JS at end of `<body>`** (in order):
1. Hamburger toggle
2. Scroll reveal via `IntersectionObserver` (`.reveal` → `.visible`)
3. Today highlight in hours table (`data-day` attribute, `0`=Sunday)
4. Back-to-top button
5. Lightbox (gallery items carry `data-lb-src` / `data-lb-alt`)
6. YouTube facade (`.yt-facade[data-video-id]` → replaced with `<iframe>` on click)

**Performance constraints** (mobile PageSpeed target 78+):
- `fixCzechTypo` was removed from JS — any JS that iterates/mutates all DOM text nodes after FCP will tank TBT. Do not re-introduce it.
- `loading="lazy"` on all images except the hero (`loading="eager" fetchpriority="high"`).
- Google Fonts loaded async via `onload` trick + `<noscript>` fallback.

## External integrations

- **Reservio** booking: `https://notorious-barbershop-kyjov.reservio.com/` — Ondra's staff link ends `…/2f925aed-…`, Tomáš's ends `…/b94c1dc8-…`
- **YouTube embed**: `qyPDmeirfkY` (replaced by facade on click, not pre-loaded)
- **Google Maps embed**: `#contact` section

## Deploy

Vercel auto-deploys `master` → `https://www.notorious-barbershop.cz/`. Cache policy in `vercel.json`: images get `max-age=31536000, immutable`; HTML gets `no-cache`. **Rename or re-content any image file when replacing it** — browsers will cache the old file for a year under the same URL.
