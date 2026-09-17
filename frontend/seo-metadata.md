# SEO & Metadata (Frontend)

**Read this first.** Search and social-share surfaces are the first impression for most users who never opened the app directly — broken metadata is invisible to the team and costly to the business. This file sets the framework-agnostic baseline. Next.js-specific implementation (Metadata API, dynamic OG image generation, sitemap/robots generation) is in [`react-next/seo-metadata.md`](../react-next/seo-metadata.md).

**Applies to:** any page intended to be indexed or shared.

---

## Mandatory

### Baseline meta tags
Every indexable page ships:

```html
<title>Concise, unique, ≤ 60 characters</title>
<meta name="description" content="Unique, ≤ 155 characters, summarizes the page." />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<link rel="canonical" href="https://example.com/current-path" />
```

- **Title and description are unique per page** — duplicate titles across a site are one of the most common, most damaging SEO defects.
- **Canonical URL** is set explicitly on every page, including the "default" version, to prevent duplicate-content penalties from query params, trailing slashes, or `www`/non-`www` variants.
- Pages that should **not** be indexed (internal tools, staging, user-specific dashboards) carry `<meta name="robots" content="noindex, nofollow" />` — don't rely on `robots.txt` alone for pages that might still get linked to externally.

### Open Graph & Twitter Card
Required on any page users might share (marketing pages, articles, product pages):

```html
<meta property="og:title" content="..." />
<meta property="og:description" content="..." />
<meta property="og:image" content="https://example.com/og/page-slug.png" />
<meta property="og:url" content="https://example.com/current-path" />
<meta property="og:type" content="website" />
<meta name="twitter:card" content="summary_large_image" />
```

- `og:image` is an **absolute URL**, minimum 1200×630px, under 5MB.
- Test every shareable page template with a link-preview debugger before shipping (e.g. a social platform's own card validator).

### Structured data (JSON-LD)
Use JSON-LD (not microdata/RDFa) for structured data, scoped to what's actually on the page — don't mark up content invisible to users:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "...",
  "datePublished": "2026-09-17"
}
</script>
```

- Pick the schema.org type that matches the content (`Article`, `Product`, `Organization`, `BreadcrumbList`, etc.).
- Validate with a structured-data testing tool before merge — invalid JSON-LD is silently ignored by crawlers, so it fails quietly.

---

## Recommended (opt-in)

- **`hreflang` tags** once i18n is enabled (see [`i18n.md`](i18n.md)), pointing to the equivalent page in each supported locale.
- **Dynamic OG image generation** per page (product image, article headline overlay) rather than one static fallback image — high-value for content/marketing sites, low priority for internal tools.

---

## Anti-Patterns (do not ship)

- Same `<title>`/description on every page (or copied from a template with placeholders left in).
- Missing or relative-only canonical URLs.
- `og:image` behind auth or as a relative path (crawlers won't resolve it).
- Structured data describing content that isn't actually rendered on the page.
- Indexable pages with no `robots` decision made either way — treat it as an explicit choice, not a default.

---

## Quick Reference

```
✓ Unique title (≤60) + description (≤155) per page, explicit canonical URL
✓ noindex on internal/user-specific pages
✓ OG + Twitter Card tags with absolute og:image URL on shareable pages
✓ JSON-LD structured data validated before merge
✗ No duplicate/templated titles
✗ No relative or missing canonical URLs
✗ No structured data describing off-page content
```

---
*Section version: 0.1 — initial draft*
