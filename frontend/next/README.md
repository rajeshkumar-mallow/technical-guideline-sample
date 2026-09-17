# Next.js Guidelines

Next.js (App Router)-specific rules. Builds on [`../react/`](../react/README.md) — read that first for framework-agnostic React conventions (components, hooks, state management) — plus [`../../shared/`](../../shared/README.md), [`../../js/`](../../js/README.md), and [`../`](../README.md).

**Version baseline:** Next.js 15+, React 19+, App Router. *(Update this line when the project's baseline changes — see [Versioning & Release Process](../../shared/versioning-release-process.md).)*

**Stable vs. experimental:** fast-moving framework features (e.g. Partial Prerendering, the React Compiler, `"use cache"`) are called out explicitly in [Rendering Strategies](rendering-strategies.md) as experimental. Treat them like an opt-in add-on, not baseline — adopting one in production requires an ADR and gets revisited each time Next/React is upgraded.

| File | Covers |
|------|--------|
| [Routing & Data Fetching](routing-data-fetching.md) | App Router, data fetching, loading/error states |
| [Rendering Strategies](rendering-strategies.md) | SSR/SSG/ISR/RSC, Server vs. Client Component boundary, stable vs. experimental |
| [Performance](performance.md) | Image/font optimization, caching, code splitting |
| [Security](security.md) | Server Action validation, env var leakage, RSC data exposure |
| [Authentication](auth.md) | Middleware route protection, session handling in RSC vs. client |
| [Internationalization](i18n.md) | App Router locale routing, middleware detection |
| [Forms & Validation](forms-validation.md) | Server Actions, `useActionState`, progressive enhancement |
| [Error Boundaries](error-boundaries.md) | error.tsx/global-error.tsx/not-found.tsx file conventions |
| [API Integration](api-integration.md) | Route handlers, Server Actions, error contracts |
| [Testing](testing.md) | Mocking the App Router, Server Component/Action testing, Playwright |
| [Analytics & Tracking](analytics-tracking.md) | Script strategy, route-change tracking |
| [SEO & Metadata](seo-metadata.md) | Metadata API, sitemap/robots, dynamic OG images |

Component architecture, state management, hooks, and the base error-boundary/testing patterns are framework-agnostic — see [`../react/`](../react/README.md); this folder only covers what Next.js adds on top.

Back to [`frontend/`](../README.md) · [root guideline](../../README.md).
