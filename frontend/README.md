# Frontend Guidelines

Browser/DOM-specific rules — applies to any frontend stack, regardless of framework. Assumes [`../shared/`](../shared/README.md) and [`../js/`](../js/README.md) as a baseline. Pick your framework's subfolder for framework-specific rules that extend these.

| File | Covers |
|------|--------|
| [HTML & CSS](html-css.md) | Semantic HTML, CSS methodology/naming |
| [Styling Foundations](styling-foundations.md) | CSS reset, preprocessor choice, design tokens, UI library selection |
| [Design to Code](design-to-code.md) | Figma token sync, component handoff, style guide/Storybook |
| [Accessibility](accessibility.md) | WCAG target, ARIA, keyboard/focus, reduced-motion, axe-core |
| [Performance](performance.md) | Bundle size, lazy loading, Core Web Vitals, images/fonts |
| [Security](security.md) | XSS, CSP, sanitization, dependency scanning |
| [Authentication](auth.md) | Token storage, CSRF, protected routes |
| [PWA & Service Workers](pwa-service-worker.md) | Caching, offline support, client-side storage |
| [Realtime & WebSockets](realtime-websockets.md) | WebSocket/SSE patterns, reconnection, handshake auth |
| [Internationalization](i18n.md) | Locale structure, pluralization, RTL, formatting |
| [Forms & Validation](forms-validation.md) | Native form handling, accessible errors, validation patterns |
| [Error Monitoring](error-monitoring.md) | Client error tracking setup |
| [Responsive Design](responsive-design.md) | Mobile-first, breakpoints, touch ergonomics |
| [Tooling & Build](tooling-build.md) | Bundler/package manager, build configuration |
| [Analytics & Tracking](analytics-tracking.md) | Script loading, event naming, consent |
| [SEO & Metadata](seo-metadata.md) | Meta tags, Open Graph, structured data |
| [API Resilience](api-resilience.md) | Client-side fetch timeouts, retry/backoff, error classification |
| [Browser & Device Support Matrix](browser-support-matrix.md) | Officially supported browsers/versions |

## Framework-specific subfolders

| Folder | Status | Covers |
|---|---|---|
| [`react/`](react/README.md) | **Active** | React fundamentals — framework-agnostic, standalone or under any meta-framework |
| [`next/`](next/README.md) | **Active** | Next.js (App Router) — builds on `react/` |
| [`preact/`](preact/README.md) | Reserved | Not yet written — likely reuses much of `react/` directly (API-compatible via `preact/compat`) |
| [`stimulus/`](stimulus/README.md) | Reserved | Not yet written — add when a project needs it |
| [`vue/`](vue/README.md) | Reserved | Not yet written — add when a project needs it |

A Next.js project follows `shared/` + `js/` + `frontend/` + `frontend/react/` + `frontend/next/`. A standalone React project (e.g. Vite + React Router) follows `shared/` + `js/` + `frontend/` + `frontend/react/` alone. A project on a non-React framework follows `shared/` + `js/` + `frontend/` + that framework's subfolder once it's written.

Back to [root guideline](../README.md).
