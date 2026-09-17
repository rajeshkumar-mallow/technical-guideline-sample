# React / Next.js Guidelines

React- and Next.js-specific rules. Builds on [`../shared/`](../shared/README.md) and [`../frontend/`](../frontend/README.md) — read those first; this folder covers what's specific to the framework.

**Version baseline:** Next.js 15+, React 19+, App Router. *(Update this line when the project's baseline changes — see [Versioning & Release Process](../shared/versioning-release-process.md).)*

**Stable vs. experimental:** fast-moving framework features (e.g. Partial Prerendering, the React Compiler, `"use cache"`) are called out explicitly in [Rendering Strategies](rendering-strategies.md) as experimental. Treat them like an opt-in add-on, not baseline — adopting one in production requires an ADR and gets revisited each time Next/React is upgraded.

| File | Covers |
|------|--------|
| [Components & Architecture](components-architecture.md) | Folder conventions, composition, props/typing |
| [State Management](state-management.md) | Local vs. global state, context, server state |
| [Hooks](hooks.md) | Custom hook conventions, rules of hooks |
| [Routing & Data Fetching](routing-data-fetching.md) | App Router, data fetching, loading/error states |
| [Rendering Strategies](rendering-strategies.md) | SSR/SSG/ISR/RSC, server vs. client components, stable vs. experimental |
| [Performance](performance.md) | Image/font optimization, caching, code splitting |
| [Security](security.md) | Server Action validation, env var leakage, RSC data exposure |
| [Authentication](auth.md) | Middleware route protection, session handling in RSC vs. client |
| [Internationalization](i18n.md) | App Router locale routing, middleware detection |
| [Forms & Validation](forms-validation.md) | react-hook-form + zod, Server Actions |
| [Error Boundaries](error-boundaries.md) | Error boundary components, error.tsx/global-error.tsx |
| [API Integration](api-integration.md) | Route handlers, Server Actions, error contracts |
| [Testing](testing.md) | RTL, mocking the Next router, Playwright |
| [Analytics & Tracking](analytics-tracking.md) | Script strategy, route-change tracking |
| [SEO & Metadata](seo-metadata.md) | Metadata API, sitemap/robots, dynamic OG images |

Back to [root guideline](../README.md).
