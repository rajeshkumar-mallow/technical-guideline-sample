# Performance

**Read this first.** Performance is a user-facing correctness concern, not a post-launch optimization pass. Budgets and targets below are checked before merge, not discovered in production.

**Applies to:** All frontend surfaces (any framework). React/Next-specific mechanisms (Image/Font components, RSC streaming) live in [react-next/performance.md](../react-next/performance.md), which extends this file.

---

## Mandatory

### Bundle size budgets
- Define an explicit **JS budget per route/entry point** (e.g. 170KB gzipped for an initial route bundle) in CI, enforced via bundle-analyzer or a size-limit tool. A PR that breaks the budget fails CI unless justified with an ADR.
- Audit bundle composition regularly (`vite-bundle-visualizer`, `webpack-bundle-analyzer`, or Next's built-in analyzer) — a single unexpectedly large dependency is the most common budget-breaker.
- Prefer named imports from libraries that support tree-shaking (`import { debounce } from 'lodash-es'`, never `import _ from 'lodash'`).

### Code-splitting & lazy loading
- **Route-level code-splitting is mandatory** — each route ships only the JS it needs, not the whole app, via dynamic `import()`.
- Split heavy, below-the-fold, or conditionally-rendered components (rich text editors, charts, modals) with dynamic import rather than bundling them into the initial payload:

```javascript
const ChartPanel = lazy(() => import('./ChartPanel'));

<Suspense fallback={<ChartSkeleton />}>
  <ChartPanel data={data} />
</Suspense>
```

- Lazy-load below-the-fold images and iframes with native `loading="lazy"`; never lazy-load the LCP (largest contentful paint) image.
- Prefetch likely-next-route chunks on hover/viewport-intersection for perceived-instant navigation, but never at the cost of blocking the current route's budget.

### Core Web Vitals — targets (75th percentile, field data)

| Metric | Target | What it measures |
|--------|--------|-------------------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | Time to render the largest visible content element |
| **INP** (Interaction to Next Paint) | ≤ 200ms | Responsiveness to user interaction |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | Visual stability — unexpected layout jumps |

- Track these via field data (Chrome UX Report / RUM), not just lab scores — lab tools (Lighthouse) catch regressions early but field data is what ships to users. Tie to [Analytics & Tracking](analytics-tracking.md) for RUM wiring.
- A PR that regresses a Lighthouse CI score below the project's set threshold fails CI.

### Images
- Serve modern formats (**AVIF/WebP** with a fallback) and correctly sized responsive images (`srcset`/`sizes`), never a single oversized asset scaled down by CSS.
- Always set explicit `width`/`height` (or `aspect-ratio`) on images and embeds to prevent CLS from late-loading assets.
- The LCP image (usually a hero image) is **not** lazy-loaded and is preloaded when it's render-blocking-critical: `<link rel="preload" as="image" href="...">`.

### Fonts
- Self-host fonts or use a performant font CDN with `preconnect`; avoid render-blocking third-party font loaders that block first paint.
- `font-display: swap` (or `optional` for non-critical fonts) so text renders immediately in a fallback font rather than blocking.
- Subset fonts to the character sets actually used; load only the weights/styles in use.
- Preload the critical above-the-fold font file.

---

## Recommended (opt-in)

- Resource hints (`preconnect`, `dns-prefetch`) for critical third-party origins (analytics, payment iframe, font CDN).
- Service-worker-based caching for repeat-visit performance — see [PWA & Service Workers](pwa-service-worker.md).
- Virtualization (`react-window`/`@tanstack/virtual`) for long lists (hundreds+ of rows) instead of rendering everything.

---

## Anti-Patterns (do not ship)

- Importing an entire library for one function (`import _ from 'lodash'` for `debounce`).
- Unsized images/embeds causing layout shift on load.
- Blocking first paint on a synchronous third-party script in `<head>` with no `async`/`defer`.
- Lazy-loading the LCP image.
- Shipping a route's entire feature set (including rarely-used modals/editors) in the initial bundle instead of splitting.
- Ignoring a failing bundle-budget or Lighthouse CI gate by disabling the check instead of fixing the regression.

## Quick Reference

```
✓ Explicit JS budget per route, enforced in CI
✓ Route-level + heavy-component code-splitting via dynamic import()
✓ LCP ≤ 2.5s · INP ≤ 200ms · CLS ≤ 0.1 (field data, p75)
✓ Responsive AVIF/WebP images, explicit width/height, LCP image preloaded not lazy
✓ font-display: swap/optional, subsetted, critical font preloaded
✗ No whole-library imports for single functions
✗ No unsized media causing CLS
✗ No lazy-loaded LCP image
```

---
*Section version: 0.1 — initial draft*
