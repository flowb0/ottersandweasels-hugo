# Otters and Weasels — Hugo + Cloudflare Pages

Source repo for [ottersandweasels.com](https://ottersandweasels.com/) — a music
blog from Vienna ("we write about music n stuff"). Migrated from WordPress to
Hugo, deployed via Cloudflare Pages.

## Tech Stack

| Layer | Tool |
|-------|------|
| SSG | Hugo (extended) |
| Theme | `themes/ottersandweasels` (custom) |
| CSS | Vanilla CSS — fingerprinted via Hugo asset pipeline (`assets/css/style.css`) |
| Hosting | Cloudflare Pages |
| Language | English |

## Build & Dev

```bash
hugo server -D      # development
hugo --minify       # production build
```

No Node build — pure Hugo.

## Structure notes

- **CSS lives in `assets/css/style.css`** (project root, not the theme) so Hugo
  can fingerprint it. The theme's `baseof.html` loads it via
  `resources.Get | minify | fingerprint` with SRI. Never link `/css/style.css`
  directly — `static/_headers` marks `/css/*` immutable for a year.
- `layouts/404.html` — branded error page.
- `layouts/robots.txt` — custom robots with AI-crawler allows (GPTBot,
  ClaudeBot, PerplexityBot, OAI-SearchBot); `static/llms.txt` for GEO.
- `layouts/_default/_markup/render-image.html` — content images get
  `loading="lazy"` + `width`/`height` from the file at build time (CLS guard).
- `static/_headers` — security headers (HSTS, nosniff, X-Frame-Options,
  Referrer-Policy, Permissions-Policy) + long caching for CSS/images.
- OpenGraph via Hugo's internal template; the site-wide `images` param in
  `hugo.toml` is the og:image fallback.

## Images

Photos ≤1600 px, JPEG quality ~82, target <300 KB. The WordPress-era archive
was batch-compressed in July 2026 (268 MB → 100 MB, in place, same filenames).
Compress before committing:

```bash
magick input.jpg -strip -resize '1600x>' -quality 82 output.jpg
```

## Changelog

### 2026-07-05 — Site hardening (freshestweb audit learnings)
- CSS moved to asset pipeline (was unfingerprinted `/css/style.css`)
- `canonical` link + og:image fallback added (site had no og:image at all)
- Batch-compressed 83 images >500 KB: **268 MB → 100 MB**
- Branded 404, custom robots.txt (AI crawlers), `llms.txt`, `_headers`,
  render-image hook, `enableRobotsTXT = true`
