# Error Boundaries

**Read this first.** Unhandled render errors should never blank the whole app. This file governs the underlying React error-boundary mechanism and how it feeds the tracker configured in [`../error-monitoring.md`](../error-monitoring.md). Next.js's file-based conventions (`error.tsx`, `global-error.tsx`, `not-found.tsx`) build on this same mechanism — see [`../next/error-boundaries.md`](../next/error-boundaries.md).

**Applies to:** any React codebase, around any component boundary with risky rendering (third-party widgets, data-dependent UI that can throw).

---

## Mandatory

### Wrap risky subtrees in an error boundary
Use the **`react-error-boundary`** package rather than hand-rolling a class component — it gives you `FallbackComponent`, `onError`, and `resetKeys` without re-deriving `componentDidCatch` boilerplate. Wrap any subtree that renders third-party widgets, complex data-dependent UI, or anything else that can throw during render.

```tsx
import { ErrorBoundary } from "react-error-boundary";
import { reportError } from "@/lib/error-monitoring"; // see frontend/error-monitoring.md

function DashboardErrorFallback({ error, resetErrorBoundary }: { error: Error; resetErrorBoundary: () => void }) {
  return (
    <div role="alert">
      <h2>Something went wrong loading the dashboard.</h2>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

export function Dashboard() {
  return (
    <ErrorBoundary FallbackComponent={DashboardErrorFallback} onError={reportError}>
      <DashboardContent />
    </ErrorBoundary>
  );
}
```

### Every caught error is reported
No error boundary swallows an error silently. Report to the tracker (`../error-monitoring.md`) with enough context (component/boundary name, relevant IDs) to triage without reproducing locally.

### Boundaries don't cover everything
An error boundary only catches errors thrown **during rendering** in its subtree — not errors in event handlers, async code, or the boundary component itself. Catch those closer to the source (`try/catch` around the risky operation) and report them the same way.

---

## Recommended (opt-in)

- **Component-level boundaries** around isolated, risky widgets (a chart library, an embed) inside a larger view that should otherwise keep working — so one broken widget doesn't take out the whole page.
- **Granular boundaries per major section** once a view grows large enough that a single page-wide boundary is too coarse. Introduce this when it's actually needed, not preemptively.
- **Retry with a limit** — if `resetErrorBoundary()` is called repeatedly without success (flaky data), fall back to a static "contact support" state instead of looping forever.

---

## Anti-Patterns (do not ship)

- Using an error boundary for an **expected** empty/not-found state — that's a conditional render, not a thrown error. Reserve boundaries for actual failures.
- Catching an error and rendering nothing (a blank div) instead of a user-facing message.
- Logging the error to `console.error` only, without forwarding to the shared error tracker.
- Putting data-fetching side effects inside a fallback component itself (it's for display and reporting, not recovery logic beyond a reset action).
- Assuming a boundary catches errors from event handlers or async callbacks — it doesn't.

## Quick Reference

```
✓ react-error-boundary around any subtree with risky rendering
✓ Every caught error reported to the tracker with context
✓ try/catch for event-handler/async errors (boundaries don't catch those)
✗ Error boundaries used for expected empty/missing states
✗ Silent catches, console-only logging
```

---
*Section version: 0.1 — initial draft*
