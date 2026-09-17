# SEO & Metadata (React/Next)

**Read this first.** Next's Metadata API replaces hand-written `<head>` tags with type-checked, composable exports. This file governs how metadata, sitemaps, and OG images are generated in the App Router — general meta-tag/structured-data principles live in [frontend/seo-metadata.md](../seo-metadata.md).

**Applies to:** Every public-facing route in a React/Next 15 App Router project.

---

## Mandatory

### Static `metadata` export for fixed pages
Any page whose title/description doesn't depend on fetched data exports a static `metadata` object. It merges with parent layouts' metadata, so shared defaults live in the root layout.

```tsx
// app/layout.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: { default: "Acme", template: "%s | Acme" },
  description: "Acme does things.",
  metadataBase: new URL("https://acme.example.com"),
};
```

```tsx
// app/pricing/page.tsx
export const metadata: Metadata = { title: "Pricing" }; // -> "Pricing | Acme"
```

### `generateMetadata` for data-dependent pages
Any page whose title/OG image depends on fetched content (a product, a blog post) uses `generateMetadata`, not a static export. Fetch data the same way the page component does — Next dedupes the request.

```tsx
// app/products/[id]/page.tsx
import type { Metadata } from "next";

export async function generateMetadata({ params }: { params: { id: string } }): Promise<Metadata> {
  const product = await getProduct(params.id);
  if (!product) return { title: "Product not found" };
  return {
    title: product.name,
    description: product.shortDescription,
    openGraph: { images: [product.imageUrl] },
  };
}

export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);
  // ...
}
```

### File-convention `sitemap.ts` and `robots.ts`
Sitemaps and robots rules are generated, not hand-maintained static files — so they stay in sync with actual routes and can pull dynamic entries (e.g. every published post) from the data layer.

```ts
// app/sitemap.ts
import type { MetadataRoute } from "next";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await getAllPublishedPosts();
  return [
    { url: "https://acme.example.com", lastModified: new Date() },
    ...posts.map((p) => ({
      url: `https://acme.example.com/blog/${p.slug}`,
      lastModified: p.updatedAt,
    })),
  ];
}
```

```ts
// app/robots.ts
import type { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: "*", allow: "/", disallow: "/admin" },
    sitemap: "https://acme.example.com/sitemap.xml",
  };
}
```

### Dynamic OG images via `next/og`
Product/article-specific social preview images are generated at request time (or statically at build time for known routes) with `ImageResponse`, not hand-designed per page.

```tsx
// app/products/[id]/opengraph-image.tsx
import { ImageResponse } from "next/og";

export default async function Image({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);
  return new ImageResponse(
    (
      <div style={{ fontSize: 64, display: "flex", padding: 40 }}>
        {product.name}
      </div>
    ),
    { width: 1200, height: 630 },
  );
}
```

Next auto-wires the resulting image into the page's `openGraph.images` — no manual `<meta>` tag needed once this file exists.

---

## Recommended (opt-in)

- **`alternates.canonical`** set explicitly on pages reachable via multiple URLs (query params, trailing slash variants) to avoid duplicate-content penalties.
- **JSON-LD structured data** via a small script component for content types search engines reward (articles, products, FAQs) — see [frontend/seo-metadata.md](../seo-metadata.md) for the general pattern.
- **`generateStaticParams`** paired with `generateMetadata` for known, finite dynamic routes (e.g. all product IDs) so metadata is pre-rendered at build time instead of per-request.

---

## Anti-Patterns (do not ship)

- Hand-writing `<title>`/`<meta>` tags inside page JSX instead of using the Metadata API — loses de-duplication and merge behavior with parent layouts.
- Using a static `metadata` export on a page whose content is actually data-dependent (stale/wrong titles for dynamic content).
- Maintaining a hand-written `public/sitemap.xml` alongside real routes — it will drift.
- Designing OG images as static assets per page instead of generating them from `opengraph-image.tsx` when content is dynamic.
- Forgetting `metadataBase` — relative OG image URLs won't resolve correctly without it.

## Quick Reference

```
✓ Static metadata export for fixed pages; generateMetadata for data-dependent ones
✓ app/sitemap.ts and app/robots.ts generated from real route/data sources
✓ next/og for dynamic per-content OG images
✓ metadataBase set in the root layout
✗ Hand-written <title>/<meta> tags in page JSX
✗ Static metadata on data-dependent pages
✗ Hand-maintained public/sitemap.xml
```

---
*Section version: 0.1 — initial draft*
