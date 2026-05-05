# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static single-page website for Notorious Barbershop Kyjov. No build step, no framework, no package manager — the entire site is one `index.html` file with all CSS and JS inlined. Deploy target is Vercel (root dir: `.`).

## Development

Open `index.html` directly in a browser or use any static file server:

```bash
npx serve .
# or
python -m http.server 8080
```

There are no tests, no linting config, and no CI pipeline.

## Architecture

Everything lives in `index.html`:

1. **CSS custom properties** (`--bg-*`, `--color-*`, `--font-*`) defined in `:root` — always use these variables instead of hardcoded values.
2. **Sections** (in DOM order): `#site-header` → `#hero` → `#about` → `#services` → `#voucher` → `#team` → `#gallery` → `#contact` → `#footer`
3. **Scroll reveal** — elements with `.reveal` (+ `.reveal-delay-1..4`) animate in via `IntersectionObserver` at page bottom.
4. **Gallery lightbox** — `#lightbox` overlay with prev/next navigation, driven by JS at end of `<body>`.
5. **Hamburger menu** — mobile breakpoint (`max-width: 900px`), toggles `.open` on `#navLinks` via `IntersectionObserver`-free click handler.

## Header

The header (`#site-header`) is `position: relative` (not sticky). It has:
- `.header-bg` — full-bleed `bg-street.jpg` with `rgba(18,16,14,0.82)` dark overlay via `::after`
- `.header-logo` — centered logo block above the nav row
- `.header-nav` — flex row with centered nav links, absolute-positioned social icons (right), and hamburger (left, mobile only)

## External integrations

- **Reservio** booking: `https://notorious-barbershop-kyjov.reservio.com/` — used in CTA buttons throughout
- **YouTube embed**: `#about` section (`https://www.youtube.com/embed/qyPDmeirfkY`)
- **Google Maps embed**: `#contact` section
- **Fonts**: Playfair Display + DM Sans via Google Fonts

## Deploy

Vercel auto-deploys from `master` branch on GitHub (`CarbonLikeADiamond/barber2`). Cache and security headers are set in `vercel.json` — images get 1-year immutable cache, HTML gets no-cache. Use `vercel` CLI for logs and deploy info.
