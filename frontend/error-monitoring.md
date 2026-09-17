# Error Monitoring (Frontend)

**Read this first.** An error a user hits silently is a bug you'll never fix. Client-side error monitoring exists to close that loop — but the same pipeline that captures a stack trace can just as easily capture a password if you're not deliberate about what gets logged.

**Applies to:** any frontend codebase. Ties to [API Resilience](api-resilience.md) for handling failed requests gracefully, and to [Privacy & Data Compliance](../shared/privacy-compliance.md) for what may leave the browser at all.

---

## Mandatory

### Global handlers
- Register a global handler for **uncaught exceptions** (`window.onerror` / `error` event) and **unhandled promise rejections** (`unhandledrejection`) at app boot, before any feature code runs.
- These are a safety net, not the primary mechanism — most errors should be caught closer to the source (try/catch around risky operations, framework error boundaries — see [`../react-next/error-boundaries.md`](../react-next/error-boundaries.md) for React specifics) so you get better context than a bare global handler provides.

```ts
window.addEventListener("error", (event) => {
  reportError(event.error ?? event.message, { source: "window.onerror" });
});

window.addEventListener("unhandledrejection", (event) => {
  reportError(event.reason, { source: "unhandledrejection" });
});
```

### Error tracking setup (Sentry-style)
- Use a hosted error tracker (Sentry or equivalent) rather than only console logging — production console output is not monitored by anyone.
- Upload **source maps** on every production build so stack traces resolve to real file/line, not minified gibberish. Do not ship source maps as publicly served files — upload them directly to the tracker and exclude them from the deployed bundle.
- Tag errors with **release version** and **environment** so a regression can be bisected to the deploy that introduced it.
- Set a **sample rate** for high-traffic apps to control cost, but never sample below 100% for unhandled exceptions in early rollout phases.

### What NOT to log
Filter these before an event ever leaves the browser — the same discipline the Rails guideline applies server-side (its §11 log filtering) applies here, just enforced client-side:

| Never send to the error tracker | Why |
|---|---|
| Passwords, tokens, session/cookie values | Credential leak via the monitoring vendor |
| Full request/response bodies containing PII | Turns an error tracker into an unintended PII store |
| Auth headers (`Authorization`, `Cookie`) | Same as tokens above |
| Full form field values on validation errors | Log the field name that failed, not its value |

- Use the tracker's built-in **scrubbing/`beforeSend` hook** to strip known-sensitive keys as a backstop, in addition to not passing them in the first place.

```ts
Sentry.init({
  beforeSend(event) {
    delete event.request?.headers?.Authorization;
    delete event.request?.headers?.Cookie;
    return event;
  },
});
```

### Context worth including
- Correlation/request id (if the backend issues one), current route, app version, and a breadcrumb trail of recent user actions — these turn a bare stack trace into something reproducible.

## Recommended (opt-in)

- **Session replay** (Sentry Replay or similar) for high-value flows, with aggressive input/text masking enabled by default — treat this as higher privacy risk than error capture alone and record the decision via ADR.
- **Real User Monitoring (RUM)** alongside error tracking to correlate errors with performance regressions — ties to [Performance](performance.md).

## Anti-Patterns (do not ship)

- Logging raw request/response bodies "for debugging" without redaction.
- Shipping source maps as publicly accessible files on the production origin.
- No release/version tagging, making it impossible to tell which deploy introduced a spike.
- Swallowing errors (`catch {}`) instead of reporting them — a silent catch is worse than no catch.
- Relying on `console.error` alone in production with no aggregation or alerting.

## Quick Reference

```
✓ Global window.onerror + unhandledrejection handlers at boot
✓ Hosted error tracker with uploaded (not publicly served) source maps
✓ Release/environment tagging on every event
✓ beforeSend scrubbing of auth headers, tokens, PII
✓ Breadcrumbs + correlation id for reproducibility
✗ Raw request/response bodies or credentials sent to the tracker
✗ Publicly served source maps
✗ Silent catch blocks
```

---
*Section version: 0.1 — initial draft*
