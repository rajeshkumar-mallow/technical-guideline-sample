# API Resilience (Frontend)

**Read this first.** Every client-side call to a backend or third-party API can fail, hang, or time out. These patterns are mandatory for any outbound `fetch`/HTTP call so failures degrade gracefully instead of hanging the UI or silently losing user input.

**Applies to:** All frontend projects (any framework). React/Next server-side outbound calls (Route Handlers, Server Actions) follow the equivalent server-side patterns in [react-next/api-integration.md](../react-next/api-integration.md); this file governs calls made **from the browser**.

---

## Mandatory

### Timeouts — every call
No unbounded `fetch`. Every outbound call sets an explicit timeout via `AbortController`:

```typescript
async function fetchWithTimeout(url: string, options: RequestInit = {}, timeoutMs = 8000) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), timeoutMs);
  try {
    return await fetch(url, { ...options, signal: controller.signal });
  } finally {
    clearTimeout(timer);
  }
}
```

### Error classification

| Class | Examples | Action |
|-------|----------|--------|
| **Transient** | Network error, timeout/abort, HTTP 408/5xx | Retry with backoff |
| **Rate-limited** | HTTP 429 | Honor `Retry-After`; back off; surface a "slow down" state if repeated |
| **Permanent** | HTTP 400/422 validation, 401/403 auth | **Do not retry** — surface the error to the user or redirect (401 → re-auth) |

A shared client-side HTTP wrapper classifies the response/error once, so every call site handles the same three outcomes consistently rather than re-implementing status-code logic per feature.

### Retry with backoff + jitter
- Retries apply **only to transient and rate-limited failures**, never to permanent ones (a 422 will fail identically on retry).
- Bounded max attempts (e.g. 3), **exponential backoff with jitter** so a burst of client retries doesn't itself become a thundering herd against a recovering backend:

```typescript
async function retryWithBackoff<T>(fn: () => Promise<T>, maxAttempts = 3): Promise<T> {
  let attempt = 0;
  while (true) {
    try {
      return await fn();
    } catch (err) {
      attempt += 1;
      if (attempt >= maxAttempts || !isRetryable(err)) throw err;
      const backoff = 2 ** attempt * 200;
      const jitter = Math.random() * 100;
      await new Promise((r) => setTimeout(r, backoff + jitter));
    }
  }
}
```

- **Mutations are not blindly retried** unless they're idempotent or carry an idempotency key the backend honors — a retried `POST /orders` must not double-submit. Prefer retrying only `GET`s automatically; gate mutation retries behind an explicit idempotency contract with the backend.

### User-facing state — never a dead end
- A failed request always resolves to a visible state: an inline error with a retry action, a toast, or a redirect — never a silently stuck spinner or a blank screen.
- Distinguish **"retry available"** (transient) from **"fix your input"** (validation) from **"you're not allowed"** (auth) in the copy shown, not just in a generic "Something went wrong."
- Loss of network mid-form should preserve the user's input (local draft state) rather than discarding it on failure.

### Telemetry
- Log attempt count, final outcome, and latency for outbound calls to the error-monitoring pipeline (see [Error Monitoring](error-monitoring.md)) — without logging request/response bodies that may carry PII (see [Privacy & Data Compliance](../shared/privacy-compliance.md)).

---

## Recommended (opt-in)

- A small circuit-breaker for a flaky third-party client-side SDK (e.g. a chat widget) that stops attempting reconnects after N consecutive failures, with a cooldown before retrying — same idea as the server-side circuit breaker pattern, scaled down.
- React Query / SWR (or equivalent) as the retry/cache layer in a React/Next app rather than hand-rolling the wrapper above per project — see [react-next/state-management.md](../react-next/state-management.md) and [react-next/api-integration.md](../react-next/api-integration.md).

---

## Anti-Patterns (do not ship)

- `fetch` with no timeout — a hung request leaves the UI in a permanent loading state.
- Retrying a 422/401/403 as if it were transient.
- Retrying a non-idempotent mutation without an idempotency key, risking a duplicate order/submission.
- A generic "Something went wrong" with no retry path and no distinction between error classes.
- Losing user-entered form data when a submit request fails.

## Quick Reference

```
✓ Every outbound call has an explicit timeout (AbortController)
✓ Transient/rate-limited → retry with backoff+jitter (bounded attempts)
✓ Permanent (4xx validation/auth) → never retried, surfaced to user
✓ Mutation retries only with idempotency guarantee
✓ Failure always resolves to a visible, actionable user state
✗ No unbounded fetch · no blind retry of permanent errors
✗ No silent stuck-spinner or blank-screen failure states
✗ No discarding user input on a failed submit
```

---
*Section version: 0.1 — initial draft*
