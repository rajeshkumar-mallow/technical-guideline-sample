# Error Boundaries

**Read this first.** Unhandled render errors should never blank the whole app. This file governs how errors are caught and presented at the component and route-segment level in React 19 / Next.js 15 (App Router), and how they feed the tracker configured in [frontend/error-monitoring.md](../frontend/error-monitoring.md).

**Applies to:** Every route segment and every component boundary around risky rendering (third-party widgets, data-dependent UI).

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
Call `notFound()` from `next/navigation` when a lookup fails (not a thrown error) — it renders the nearest `not-found.tsx` with a proper 404 status, instead of surfacing a 500 via `error.tsx`.

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

### Every caught error is reported
No `error.tsx` or manual `<ErrorBoundary>` swallows an error silently. Report to the tracker (`frontend/error-monitoring.md`) with enough context (`digest`, boundary name, relevant IDs) to triage without reproducing locally.

---

## Recommended (opt-in)

- **Component-level boundaries** (`react-error-boundary` package) around isolated, risky widgets (a chart library, an embed) inside a page that should otherwise keep working — so one broken widget doesn't take out the whole segment.
- **Granular `error.tsx` per nested segment** once a route tree grows large enough that a single dashboard-wide error boundary is too coarse (e.g. one per major tab). Introduce this when it's actually needed, not preemptively.
- **`reset()` with a retry limit** — if `reset()` is called repeatedly without success (flaky data), fall back to a static "contact support" state instead of looping forever.

---

## Anti-Patterns (do not ship)

- Using `error.tsx` for expected "not found" states — that's what `notFound()` + `not-found.tsx` are for; reserve `error.tsx` for actual failures.
- Catching an error and rendering nothing (a blank div) instead of a user-facing message.
- Skipping `global-error.tsx` — without it, a root layout crash shows the framework's default unstyled error screen in production.
- Logging the error to `console.error` only, without forwarding to the shared error tracker.
- Putting data-fetching side effects inside `error.tsx` itself (it's for display and reporting, not recovery logic beyond `reset()`).

## Quick Reference

```
✓ error.tsx per data-dependent route segment
✓ Exactly one app/global-error.tsx with its own <html>/<body>
✓ notFound() + not-found.tsx for missing resources (not error.tsx)
✓ Every caught error reported to the tracker with context
✗ error.tsx used for expected empty/missing states
✗ Silent catches, console-only logging
✗ Missing global-error.tsx
```

---
*Section version: 0.1 — initial draft*
