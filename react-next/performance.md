# Performance (React/Next)

**Read this first.** Next.js gives you route-based code splitting and image/font optimization for free — most performance regressions come from opting out of those defaults, not from the framework failing to provide them. This file covers the Next-specific layer on top of the general [frontend performance](../frontend/performance.md) rules.

**Applies to:** all React/Next projects.

---

## Mandatory

### Images — `next/image`
All content images use `next/image`, not a bare `<img>`. It handles responsive sizing, lazy loading below the fold, and modern format negotiation (AVIF/WebP) automatically.

```tsx
import Image from "next/image";

<Image
  src={product.imageUrl}
  alt={product.name}      // required — never empty unless decorative (alt="")
  width={640}
  height={480}
  priority={isAboveTheFold} // only for LCP-candidate images
/>
```

- Set `priority` on the single largest above-the-fold image (usually the LCP element) — do not set it on every image, which defeats lazy loading.
- Always supply `alt`; ties to [Accessibility](../frontend/accessibility.md).
- Remote images require an allowlisted domain in `next.config.js` `images.remotePatterns` — do not disable image optimization (`unoptimized: true`) without an ADR.

### Fonts — `next/font`
Self-host fonts via `next/font/google` or `next/font/local` — never a render-blocking `<link>` to a third-party font host. This eliminates layout shift from font swap and removes an external network request.

```tsx
import { Inter } from "next/font/google";
const inter = Inter({ subsets: ["latin"], display: "swap" });
```

### Code splitting
Route-based code splitting is automatic in the App Router — each route segment ships only the JS it needs. On top of that:
- Use `next/dynamic` for heavy, below-the-fold, or conditionally-rendered client components (charting libraries, rich text editors, modals) so their JS isn't in the initial bundle.

```tsx
const RichTextEditor = dynamic(() => import("./RichTextEditor"), {
  loading: () => <EditorSkeleton />,
  ssr: false, // only when the component genuinely can't render server-side
});
```

### Caching layers
Understand which layer is serving a given response before debugging "why is this stale":

| Layer | What it caches | Controlled by |
|---|---|---|
| **Data Cache** | `fetch()` responses on the server | `cache`/`next.revalidate`/`tags` on the `fetch()` call — see [Routing & Data Fetching](routing-data-fetching.md) |
| **Full Route Cache** | The rendered HTML/RSC payload for static routes | Route's static/dynamic status (inferred, see [Rendering Strategies](rendering-strategies.md)) |
| **Router Cache** | Client-side, per-session cache of visited route segments | Automatic; invalidated by `router.refresh()` or a Server Action's `revalidatePath`/`revalidateTag` |

A stale page after a mutation is almost always a missing `revalidatePath`/`revalidateTag` call in the Server Action that performed the write, not a framework bug.

### Bundle analysis
Run `@next/bundle-analyzer` before merging any change that adds a new dependency to a client bundle, and as a recurring CI check. No PR adding a client-side dependency > 20KB gzipped ships without a stated reason (tree-shaking checked, no lighter alternative, etc.).

---

## Recommended (opt-in)

- Set a **performance budget** per route (e.g. LCP < 2.5s, JS payload < 200KB gzipped for the initial route) and track it in CI via Lighthouse CI, escalating regressions the same way test failures block merge.
- Use `loading="lazy"` semantics implicitly via `next/image` rather than manually managing IntersectionObserver-based lazy loading.

---

## Anti-Patterns (do not ship)

- Bare `<img>` tags for content images.
- Third-party `<link rel="stylesheet">` font imports instead of `next/font`.
- `priority` set on more than one image per view.
- A large client-only library (charting, editor, animation) imported at the top of a Client Component instead of via `next/dynamic`.
- `images.unoptimized: true` or `ssr: false` used as a default habit rather than a deliberate, documented exception.
- Mutating data in a Server Action without calling `revalidatePath`/`revalidateTag`, leaving stale cached pages.

---

## Quick Reference

```
✓ next/image for all content images, alt required, priority on LCP image only
✓ next/font — no external font <link> tags
✓ next/dynamic for heavy/below-the-fold client components
✓ Know your caching layer: Data Cache vs. Full Route Cache vs. Router Cache
✓ revalidatePath/revalidateTag after every mutating Server Action
✓ Bundle analyzer run before merging new client dependencies
✗ No bare <img>, no unoptimized: true without an ADR
✗ No priority spam, no undocumented ssr: false
```

See also: [Rendering Strategies](rendering-strategies.md), [Routing & Data Fetching](routing-data-fetching.md), [frontend/performance.md](../frontend/performance.md).

---
*Section version: 0.1 — initial draft*
