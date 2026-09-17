# Rendering Strategies

**Read this first.** Rendering strategy is not a one-time framework choice — it's a per-route, sometimes per-component decision. Picking the wrong one silently costs either performance (over-fetching client-side, shipping unnecessary JS) or correctness (stale data served as if fresh).

**Applies to:** every route and component in `app/`.

---

## Mandatory

### The four strategies

| Strategy | What it means | Use for |
|----------|----------------|---------|
| **SSR** (dynamic rendering) | Rendered on the server per request | Personalized, request-specific, or always-fresh content (dashboards, authenticated pages) |
| **SSG** (static rendering) | Rendered at build time, served as static HTML | Content that's identical for all users and rarely changes (marketing pages, docs) |
| **ISR** | Static rendering, revalidated on a timer or on-demand | Content that's mostly static but needs periodic freshness (product listings, blog indexes) |
| **RSC streaming** | Server Components render to a stream, sent progressively | Default for App Router — combine with Suspense so slow parts don't block fast parts |

A route's strategy in the App Router is inferred from what it does, not manually flagged: using `no-store`, reading cookies/headers, or using dynamic APIs makes a route dynamic (SSR-like) automatically; a route with no dynamic APIs is statically rendered by default. **Know which one your route is** — check with `next build` output (it lists routes as `○ Static`, `● SSG`, or `λ Dynamic`) before merging.

### Server vs. Client Components — decision rule
**Default to Server Components.** Add `"use client"` only when the component genuinely needs one of:
- Interactivity/event handlers (`onClick`, `onChange`, etc.)
- Browser-only APIs (`window`, `localStorage`, `IntersectionObserver`)
- React state/effects (`useState`, `useEffect`, `useContext`)
- A third-party library that itself requires the client

```tsx
// Server Component (default) — no directive needed
export default async function ProductList() {
  const products = await getProducts();
  return <ul>{products.map(p => <ProductCard key={p.id} product={p} />)}</ul>;
}

// Client Component — only because it needs interactivity
"use client";
export function AddToCartButton({ productId }: { productId: string }) {
  const [pending, setPending] = useState(false);
  return <button onClick={() => addToCart(productId)}>Add to Cart</button>;
}
```

Push `"use client"` **as far down the tree as possible** — mark the interactive leaf, not its server-rendered parent, so the parent's JS never ships to the browser.

### Stable vs. Experimental

Adopting anything in the right-hand column in production requires an **ADR** (rule, reason, scope, revisit trigger) and is re-reviewed at every Next.js/React minor upgrade — see [Versioning & Release Process](../shared/versioning-release-process.md).

| Stable (mandatory baseline) | Experimental (ADR required before production use) |
|---|---|
| SSR / dynamic rendering | **Partial Prerendering (PPR)** |
| SSG / static rendering | **React Compiler** (automatic memoization) |
| ISR (`revalidate`, `revalidateTag`/`revalidatePath`) | **`"use cache"` directive** |
| React Server Components + streaming | **`dynamicIO`** |
| `<Suspense>`-based streaming SSR | New/canary-flagged APIs in general |

If an experimental feature is adopted under an ADR, document the specific version it was validated against, and treat it as a candidate for removal/rollback if a Next.js upgrade changes its behavior before it stabilizes.

---

## Recommended (opt-in)

- Use `export const dynamic = "force-static"` / `"force-dynamic"` only to override inferred behavior deliberately (e.g. forcing a page static despite reading a header you know is safe to ignore) — document why in a code comment, since it overrides the framework's inference.
- Prefer ISR with tag-based `revalidateTag()` over blanket time-based `revalidate` when content changes are event-driven (e.g. "revalidate when this order updates") rather than purely time-based.

---

## Anti-Patterns (do not ship)

- `"use client"` on a whole page/layout just because one small leaf component needs interactivity.
- Fetching data client-side in a Client Component when the same component could be a Server Component with server-side data access.
- Adopting an experimental feature (PPR, React Compiler, `"use cache"`, `dynamicIO`) without an ADR.
- Assuming a route is static without checking `next build` output — shipping an accidentally-dynamic route that was meant to be cached (or vice versa).
- Passing non-serializable values (functions, class instances) as props from a Server Component to a Client Component.

---

## Quick Reference

```
✓ Server Components by default; "use client" only at the interactive leaf
✓ Know each route's rendering mode from `next build` output
✓ Every fetch() has explicit cache/revalidate intent (see Routing & Data Fetching)
✓ PPR / React Compiler / "use cache" / dynamicIO — ADR required, revisited on upgrade
✗ No client-side fetching where a Server Component would do
✗ No experimental rendering feature in production without an ADR
```

See also: [Routing & Data Fetching](routing-data-fetching.md), [Performance](performance.md), [Versioning & Release Process](../shared/versioning-release-process.md).

---
*Section version: 0.1 — initial draft*
