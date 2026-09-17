# Routing & Data Fetching

**Read this first.** The App Router's file conventions and server-first data fetching are the backbone of every Next.js route. Getting these conventions right (colocated loading/error UI, correct fetch caching) is what makes routes fast by default instead of by heroics.

**Applies to:** all routes under `app/` in every React/Next project.

---

## Mandatory

### File conventions
Every route segment is a folder under `app/` using these reserved files:

| File | Purpose |
|------|---------|
| `page.tsx` | The route's UI, must default-export a component |
| `layout.tsx` | Shared UI that persists across navigations within the segment, must render `{children}` |
| `loading.tsx` | Instant loading UI, automatically wraps `page.tsx` in a `<Suspense>` boundary |
| `error.tsx` | Client Component error boundary for the segment (must be `"use client"`) |
| `not-found.tsx` | Rendered when `notFound()` is called or a route can't be matched |
| `template.tsx` | Like `layout.tsx` but remounts on navigation — use only when you need reset-on-navigate behavior (e.g. re-triggering an enter animation) |

Route groups `(marketing)` organize without affecting the URL. Dynamic segments `[id]`, catch-all `[...slug]`, and optional catch-all `[[...slug]]` follow standard Next.js conventions — do not invent parallel ad-hoc routing via query strings when a segment fits.

### Data fetching in Server Components
Fetch data directly in Server Components with `async`/`await` — no `useEffect` + `useState` data-fetching in a Server Component, and no client-side fetch for data that's known at request/build time.

```tsx
// app/orders/[id]/page.tsx
export default async function OrderPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const order = await getOrder(id); // direct server-side data access or fetch()
  if (!order) notFound();
  return <OrderDetail order={order} />;
}
```

`fetch()` in a Server Component is auto-memoized per request-render (deduped across the tree) and carries **explicit caching intent**:

```ts
fetch(url);                                   // default: cached (Data Cache), revalidated on deploy
fetch(url, { cache: "no-store" });            // always dynamic, never cached
fetch(url, { next: { revalidate: 60 } });     // ISR-style: revalidate at most every 60s
fetch(url, { next: { tags: ["orders"] } });   // tag for on-demand revalidation via revalidateTag()
```

**Every `fetch()` call in a Server Component must set an explicit caching strategy** (`cache` or `next.revalidate`/`tags`) — do not rely on the implicit default without a documented reason; a reviewer must be able to tell from the call site whether the data is static, time-based, or always fresh.

### Parallel vs. sequential fetching
Independent data must be fetched **in parallel**, not awaited one after another:

```tsx
// Correct — both requests fire immediately
const [order, customer] = await Promise.all([getOrder(id), getCustomer(customerId)]);

// Anti-pattern — customer fetch waits on order fetch for no reason
const order = await getOrder(id);
const customer = await getCustomer(customerId);
```

Use **sequential** fetching only when the second call genuinely depends on the first's result (a request waterfall you cannot avoid) — and consider whether that dependency belongs in a single server-side query instead.

### Loading and error states
- Every route segment that fetches data on the server gets a `loading.tsx` (skeleton or spinner) — the user never stares at a blank tab during a Server Component fetch.
- Every route segment gets an `error.tsx` with a retry action (`reset()`), per [Error Boundaries](error-boundaries.md).
- Use nested `<Suspense>` boundaries around slow, non-critical data so the rest of the page can stream in first — do not wrap the entire page in one boundary when only one section is slow.

```tsx
export default function DashboardPage() {
  return (
    <>
      <SummaryCards /> {/* fast, renders immediately */}
      <Suspense fallback={<ActivitySkeleton />}>
        <ActivityFeed /> {/* slow — streams in independently */}
      </Suspense>
    </>
  );
}
```

---

## Recommended (opt-in)

- Colocate route-specific components in the segment folder (`app/orders/[id]/_components/`) using a `_` prefix to opt out of routing — keeps related UI near its route without polluting the URL space.
- Use `generateStaticParams` to pre-render known dynamic segments at build time when the param set is bounded and known ahead of time (e.g. product slugs); fall back to on-demand rendering for the long tail.
- For client-triggered refetches (e.g. "Refresh" button), prefer Server Actions + `revalidatePath`/`revalidateTag` over client-side `fetch` + local state, keeping caching centralized. See [API Integration](api-integration.md).

---

## Anti-Patterns (do not ship)

- Fetching in a Client Component (`useEffect`) for data that's available server-side at request time.
- `fetch()` calls with no explicit `cache`/`revalidate` intent on data that's clearly static or clearly always-fresh.
- Sequential `await` chains for independent data (accidental waterfalls).
- Business/query logic duplicated across `page.tsx` files instead of extracted into a shared data-access function.
- Missing `loading.tsx` on a route that performs a non-trivial server fetch.

---

## Quick Reference

```
✓ Reserved files: page/layout/loading/error/not-found/template
✓ Fetch server-side by default — no client useEffect fetch for initial data
✓ Every fetch() has explicit cache/revalidate/tags intent
✓ Promise.all for independent fetches — no accidental waterfalls
✓ loading.tsx + error.tsx on every data-fetching route segment
✓ Suspense boundaries around slow, non-critical sections only
✗ No client-side fetch for request-time data
✗ No unbounded default-cache fetch() without a documented reason
```

See also: [Rendering Strategies](rendering-strategies.md), [API Integration](api-integration.md), [Error Boundaries](error-boundaries.md).

---
*Section version: 0.1 — initial draft*
