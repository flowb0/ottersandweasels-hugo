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

## SEO & GEO

- **Head** (`layouts/partials/seo-meta.html`, called from the project `baseof.html` override): title
  `<Post> – Otters and Weasels`, description (front matter `description` → cleaned post summary → category line →
  `params.seoDescription`), robots, canonical, hreflang, OG/Twitter with the post `image` (size + alt), `article:*`.
  Replaces Hugo's internal OpenGraph template.
- **JSON-LD** (`layouts/partials/schema.html`, one `@graph` via `jsonify`): WebSite + Organization (founded 2013,
  Vienna, Spotify) everywhere; BlogPosting with the named author (Person) on posts; BreadcrumbList via category.
- **One H1 per page**: home H1 = header logo (`layouts/partials/header.html`, `.logo-h1` keeps the layout);
  `# …` in post bodies renders as H2 (`layouts/_default/_markup/render-heading.html`).
- **Tag archives** (268) are `noindex, follow` and left out of the sitemap (`layouts/sitemap.xml`).
- **Old WordPress URLs** (`/<slug>/`, `/category/…`, `/tag/…`, `/author/…`, `/feed/`, `/wp-content/uploads/…`)
  301 to the Hugo paths via `static/_redirects` — generated from content plus the Wayback Machine URL list,
  13 renamed slugs mapped by hand. New posts need no entry.
- **robots.txt** welcomes all search and AI crawlers; `llms.txt` lists every post by category — update it for new posts.
- Static pages need a `description` in front matter. The German privacy page sets `htmlLang: "de"`.

## Changelog

### 2026-11-21 — SEO/GEO pass

- Old WordPress URLs returned 404 → 248 static + 9 pattern 301s in `static/_redirects`.
- Every post had the site tagline as description → post summaries; own descriptions for the static pages.
- Own head partial + JSON-LD graph; home page had no H1, 40 posts had two.
- 268 tag pages noindex + out of the sitemap (406 → 138 URLs).
- 69 absolute links to the old WordPress domain rewritten to local paths.
- robots.txt with explicit AI crawler allows; `llms.txt` regenerated with every post.

### 2026-07-05 — Site hardening (freshestweb audit learnings)
- CSS moved to asset pipeline (was unfingerprinted `/css/style.css`)
- `canonical` link + og:image fallback added (site had no og:image at all)
- Batch-compressed 83 images >500 KB: **268 MB → 100 MB**
- Branded 404, custom robots.txt (AI crawlers), `llms.txt`, `_headers`,
  render-image hook, `enableRobotsTXT = true`
