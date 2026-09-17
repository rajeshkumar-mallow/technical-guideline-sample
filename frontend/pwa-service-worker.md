# PWA & Service Workers

**Read this first.** A service worker sits between the network and every request your app makes — done well it buys resilience and offline support; done carelessly it serves stale content forever or silently breaks deploys. Treat it as production infrastructure, not a checkbox.

**Applies to:** any frontend adding offline support, install prompts, or client-side persistence.

---

## Mandatory

### Registration
- Register the service worker only in production builds (skip in dev — hot reload and SW caching fight each other).
- Always check for `navigator.serviceWorker` support before registering; PWA features are progressive enhancement, not a requirement.

```ts
if ("serviceWorker" in navigator && import.meta.env.PROD) {
  navigator.serviceWorker.register("/sw.js");
}
```

### Caching strategy — pick per resource type, not globally

| Strategy | Use for | Behavior |
|---|---|---|
| **Cache-first** | Static, versioned assets (hashed JS/CSS bundles, fonts, icons) | Serve from cache, only hit network on cache miss |
| **Network-first** | HTML documents, anything that must reflect the latest deploy | Try network, fall back to cache when offline |
| **Stale-while-revalidate** | API responses that can tolerate a moment of staleness (feed data, non-critical lists) | Serve cached copy immediately, refresh cache in background |
| **Network-only** | Auth endpoints, payment/mutation requests | Never cache — correctness matters more than availability |

Do not apply one strategy to the entire origin. Route by request type (`event.request.destination`, URL pattern) inside the `fetch` handler.

### Cache invalidation on deploy
- Version the cache name (`app-cache-v${BUILD_ID}`) and delete old-versioned caches in the service worker's `activate` event — a stale cache-first strategy without this **permanently serves an old build** to returning users.
- Call `self.skipWaiting()` + `clients.claim()` deliberately (not by default) — understand that this makes a new SW take over mid-session, which can mismatch an already-loaded HTML shell against new hashed assets. Prefer prompting the user to reload when an update is detected instead.

```ts
self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(keys.filter((k) => k !== CURRENT_CACHE).map((k) => caches.delete(k)))
    )
  );
});
```

### Offline fallback UX
- Provide a dedicated offline fallback page/state for navigation requests that fail with no cache match — never let the user hit the browser's default "no internet" error page.
- Surface connectivity state in the UI (banner/toast) rather than letting requests fail silently; ties to [API Resilience](api-resilience.md) for retry/error UX patterns.

### Client-side storage: choosing the right mechanism

| Mechanism | Use for | Avoid for |
|---|---|---|
| **`localStorage`** | Small, non-sensitive UI preferences (theme, last-used tab) | Auth tokens, PII, anything session-scoped, large/structured data |
| **`sessionStorage`** | Per-tab ephemeral state (multi-step form draft within one visit) | Anything that must survive a closed tab or be shared across tabs |
| **IndexedDB** | Structured/large data, offline datasets, queued mutations for background sync | Simple key-value flags (overkill) |

**Never store sensitive PII or auth tokens in `localStorage` or `sessionStorage`** — both are readable by any script on the page (see [Auth](auth.md) and [Security](security.md)). If offline data must include personal information, encrypt at rest in IndexedDB or avoid persisting it client-side entirely.

## Recommended (opt-in)

- **Background Sync API** to queue mutations made while offline and replay them on reconnect.
- **Install prompts** (`beforeinstallprompt`) with a deliberate, non-intrusive trigger (after meaningful engagement, not on first visit).
- A library (Workbox) instead of hand-rolled SW logic once caching rules grow past a handful of routes — reduces the chance of a subtle invalidation bug.

## Anti-Patterns (do not ship)

- Cache-first on HTML/navigation requests without a versioned invalidation strategy — users get stuck on an old build indefinitely.
- Storing auth tokens or PII in `localStorage`/`sessionStorage` "for offline access."
- A service worker with no `activate`-time cleanup of old cache versions (unbounded cache growth).
- Silently failing offline instead of showing a clear offline state.

## Quick Reference

```
✓ Register SW in production only, feature-detected
✓ Cache-first for hashed static assets, network-first for HTML, network-only for auth/payment
✓ Versioned cache name + cleanup on activate
✓ Dedicated offline fallback UX, connectivity state surfaced
✓ IndexedDB for structured/offline data; localStorage only for non-sensitive prefs
✗ Auth tokens or PII in localStorage/sessionStorage
✗ Cache-first HTML without invalidation
```

---
*Section version: 0.1 — initial draft*
