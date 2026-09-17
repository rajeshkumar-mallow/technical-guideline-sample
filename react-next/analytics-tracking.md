# Analytics & Tracking (React/Next)

**Read this first.** The App Router has no built-in page-change event (the Pages Router's `routeChangeComplete` doesn't exist here), and third-party analytics scripts loaded carelessly block rendering. This file governs script loading and route-change tracking specifically for App Router projects — general event-naming and consent rules live in [frontend/analytics-tracking.md](../frontend/analytics-tracking.md).

**Applies to:** Any analytics/tracking script (product analytics, ads pixels, A/B testing SDKs) loaded into a React/Next 15 App Router app.

---

## Mandatory

### Load third-party scripts with `next/script`, with the right `strategy`
Never drop a raw `<script>` tag in a layout — `next/script` controls loading priority so analytics never blocks the main thread or delays interactivity.

| Strategy | Use for |
|---|---|
| `afterInteractive` (default) | Most analytics (GA4, product analytics) — loads after the page is interactive |
| `lazyOnload` | Low-priority scripts (chat widgets, some ad pixels) — loads during idle time |
| `beforeInteractive` | Only for scripts required before hydration (rare — polyfills, critical anti-fraud) |

```tsx
// app/layout.tsx
import Script from "next/script";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Script
          src="https://analytics.example.com/script.js"
          strategy="afterInteractive"
        />
      </body>
    </html>
  );
}
```

### Track route changes with `usePathname`/`useSearchParams`, wrapped in `Suspense`
There's no router event to hook. Track navigations from a small Client Component that watches the pathname/search params and fires a page-view event on change. `useSearchParams` opts the component into client-side rendering, so it must be wrapped in `Suspense` to avoid de-opting the whole page.

```tsx
// components/analytics-page-view.tsx
"use client";
import { usePathname, useSearchParams } from "next/navigation";
import { useEffect } from "react";
import { trackPageView } from "@/lib/analytics"; // see frontend/analytics-tracking.md

export function AnalyticsPageView() {
  const pathname = usePathname();
  const searchParams = useSearchParams();

  useEffect(() => {
    trackPageView(`${pathname}?${searchParams.toString()}`);
  }, [pathname, searchParams]);

  return null;
}
```

```tsx
// app/layout.tsx
import { Suspense } from "react";
import { AnalyticsPageView } from "@/components/analytics-page-view";

// inside <body>:
<Suspense fallback={null}>
  <AnalyticsPageView />
</Suspense>
```

### Consent gating still applies
Loading a script via `next/script` doesn't bypass consent requirements — gate the `<Script>` render itself behind the consent state from [shared/privacy-compliance.md](../shared/privacy-compliance.md); don't fire it unconditionally and try to opt out after the fact.

```tsx
{hasAnalyticsConsent && (
  <Script src="https://analytics.example.com/script.js" strategy="afterInteractive" />
)}
```

### Server-side events for anything that must not be blockable
Events tied to business-critical funnels (purchase completed, signup completed) should also fire server-side (from the Server Action that completes the action), not rely solely on a client script that an ad-blocker can drop.

---

## Recommended (opt-in)

- **`next/third-parties`** package for well-known integrations (Google Analytics, Google Tag Manager) — it wraps `next/script` with sensible defaults; prefer it over hand-rolling the same script tag when the integration is supported.
- **Debounce/throttle** high-frequency client events (scroll depth, hover tracking) before sending — don't fire a network call per pixel of scroll.

---

## Anti-Patterns (do not ship)

- A raw `<script>` tag for analytics instead of `next/script` — loses loading-priority control and can block hydration.
- Using `useSearchParams` for route tracking without a `Suspense` boundary — forces the entire page into client-side rendering at the build.
- Firing analytics scripts before consent is known, then trying to "undo" tracking after the user declines.
- Relying only on client-side events for revenue-critical funnels that ad-blockers commonly strip.
- Re-implementing page-view tracking with `window.location` polling instead of `usePathname`/`useSearchParams`.

## Quick Reference

```
✓ next/script with strategy matched to priority (afterInteractive default)
✓ usePathname + useSearchParams in Suspense for route-change tracking
✓ Script render gated behind consent state
✓ Business-critical events also fired server-side
✗ Raw <script> tags for analytics
✗ useSearchParams without Suspense
✗ Tracking fired before consent
```

---
*Section version: 0.1 — initial draft*
