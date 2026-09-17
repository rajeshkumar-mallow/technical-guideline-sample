# Error Boundaries (Next.js)

**Read this first.** This file covers Next.js's file-based error-handling conventions, which build on the underlying React error-boundary mechanism documented in [`../react/error-boundaries.md`](../react/error-boundaries.md) — read that first for the general "every caught error is reported" principle.

**Applies to:** Every route segment in a Next.js 15 App Router project.

---

## Mandatory

### Segment-level `error.tsx`
Every route segment that renders non-trivial data-dependent UI gets an `error.tsx`. It's a Client Component, catches errors thrown during rendering **in that segment and its children**, and lets the rest of the layout (nav, sidebar) stay intact.

```tsx
// app/dashboard/error.tsx
"use client";
import { useEffect } from "react";
import { reportError } from "@/lib/error-monitoring"; // see frontend/error-monitoring.md

export default function DashboardError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    reportError(error, { digest: error.digest, boundary: "dashboard" });
  }, [error]);

  return (
    <div role="alert">
      <h2>Something went wrong loading the dashboard.</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### Root-level `global-error.tsx`
Catches errors in the **root layout itself** (where a normal `error.tsx` can't help, since it renders inside the layout it's meant to replace). It must render its own `<html>`/`<body>` — the root layout is gone when this fires. Every project has exactly one, at `app/global-error.tsx`.

```tsx
// app/global-error.tsx
"use client";
export default function GlobalError({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <html>
      <body>
        <h2>Something went wrong.</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  );
}
```

### `not-found.tsx` for missing resources
Call `notFound()` from `next/navigation` when a lookup fails (not a thrown error) — it renders the nearest `not-found.tsx` with a proper 404 status, instead of surfacing a 500 via `error.tsx`. This is the Next.js-specific instance of the general rule in [`../react/error-boundaries.md`](../react/error-boundaries.md): don't use an error boundary for an expected empty/missing state.

```tsx
// app/products/[id]/page.tsx
import { notFound } from "next/navigation";

export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);
  if (!product) notFound();
  return <ProductView product={product} />;
}
```

```tsx
// app/products/[id]/not-found.tsx
export default function ProductNotFound() {
  return <p>We couldn't find that product.</p>;
}
```

---

## Recommended (opt-in)

- **Granular `error.tsx` per nested segment** once a route tree grows large enough that a single dashboard-wide error boundary is too coarse (e.g. one per major tab). Introduce this when it's actually needed, not preemptively.
- **`reset()` with a retry limit** — if `reset()` is called repeatedly without success (flaky data), fall back to a static "contact support" state instead of looping forever.

---

## Anti-Patterns (do not ship)

- Using `error.tsx` for expected "not found" states — that's what `notFound()` + `not-found.tsx` are for; reserve `error.tsx` for actual failures.
- Skipping `global-error.tsx` — without it, a root layout crash shows the framework's default unstyled error screen in production.
- Logging the error to `console.error` only, without forwarding to the shared error tracker.
- Putting data-fetching side effects inside `error.tsx` itself (it's for display and reporting, not recovery logic beyond `reset()`).

## Quick Reference

```
✓ error.tsx per data-dependent route segment
✓ Exactly one app/global-error.tsx with its own <html>/<body>
✓ notFound() + not-found.tsx for missing resources (not error.tsx)
✗ error.tsx used for expected empty/missing states
✗ Missing global-error.tsx
```

---
*Section version: 0.1 — initial draft*
