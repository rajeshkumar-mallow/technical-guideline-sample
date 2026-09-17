# Frontend Technical Guideline

**Read this first.** This is our standing technical standard for frontend, React, and Next.js work. Follow it during design, implementation, review, and local testing. It is not tied to any single project — projects adopt it as-is and record their specifics in a short project addendum.

**Status:** A living standard, maintained centrally and versioned. **Projects do not fork or rewrite it per engagement** — they follow the mandatory sections and record project-specific decisions in their own context doc. Propose changes to the guideline itself via PR + ADR (Architecture Decision Record).

**Structure:**

- **`shared/`** — cross-cutting rules that apply regardless of whether the work is plain frontend or React/Next: git workflow, code style, TypeScript, testing philosophy, setup, approved libraries, environment/deployment, project architecture, and more.
- **`frontend/`** — framework-agnostic frontend rules: HTML/CSS, styling foundations, accessibility, performance, security, auth, PWA, i18n, forms, and more.
- **`react-next/`** — React- and Next.js-specific rules that build on `shared/` and `frontend/`: component architecture, state management, rendering strategies, routing/data fetching, and more.

A React/Next project follows **all three folders**. A plain frontend project (no React/Next) follows `shared/` + `frontend/` only.

---

## How to Use This Guideline

- The rules in `shared/`, `frontend/`, and `react-next/` apply to every project that adopts this guideline, unless an ADR documents a justified exception.
- **Do not edit these documents per project.** Each project keeps a short **project addendum** (e.g. `docs/PROJECT_CONTEXT.md`) recording what varies — stack versions, enabled add-ons, out-of-scope items, open TBCs.
- Treat this as a living standard: improvements discovered on a project flow back here via PR + ADR, so every future project benefits.
- Version baselines (e.g. which Next.js/React version is assumed) are stated in `react-next/README.md`. Fast-moving framework features are explicitly marked **stable** vs. **experimental** — see `react-next/rendering-strategies.md` — and experimental features require an ADR before adoption in production.

---

## Table of Contents

### shared/
- [Git & PR Workflow](shared/git-pr-workflow.md)
- [Code Style & Linting](shared/code-style-linting.md)
- [TypeScript Conventions](shared/typescript-conventions.md)
- [Testing Philosophy](shared/testing-philosophy.md)
- [Setup Checklist](shared/setup-checklist.md)
- [Approved Libraries](shared/approved-libraries.md)
- [Environment & Configuration](shared/environment-config.md)
- [Deployment](shared/deployment.md)
- [Project Architecture](shared/project-architecture.md)
- [Browser & Device Support Matrix](shared/browser-support-matrix.md)
- [Documentation Standards](shared/documentation-standards.md)
- [Versioning & Release Process](shared/versioning-release-process.md)
- [Feature Flags](shared/feature-flags.md)
- [Privacy & Data Compliance](shared/privacy-compliance.md)

### frontend/
- [HTML & CSS](frontend/html-css.md)
- [Styling Foundations](frontend/styling-foundations.md)
- [Design to Code](frontend/design-to-code.md)
- [JavaScript](frontend/javascript.md)
- [Accessibility](frontend/accessibility.md)
- [Performance](frontend/performance.md)
- [Security](frontend/security.md)
- [Authentication](frontend/auth.md)
- [PWA & Service Workers](frontend/pwa-service-worker.md)
- [Realtime & WebSockets](frontend/realtime-websockets.md)
- [Internationalization](frontend/i18n.md)
- [Forms & Validation](frontend/forms-validation.md)
- [Error Monitoring](frontend/error-monitoring.md)
- [Responsive Design](frontend/responsive-design.md)
- [Tooling & Build](frontend/tooling-build.md)
- [Testing Tooling](frontend/testing-tooling.md)
- [Analytics & Tracking](frontend/analytics-tracking.md)
- [SEO & Metadata](frontend/seo-metadata.md)
- [API Resilience](frontend/api-resilience.md)

### react-next/
- [Components & Architecture](react-next/components-architecture.md)
- [State Management](react-next/state-management.md)
- [Hooks](react-next/hooks.md)
- [Routing & Data Fetching](react-next/routing-data-fetching.md)
- [Rendering Strategies](react-next/rendering-strategies.md)
- [Performance](react-next/performance.md)
- [Security](react-next/security.md)
- [Authentication](react-next/auth.md)
- [Internationalization](react-next/i18n.md)
- [Forms & Validation](react-next/forms-validation.md)
- [Error Boundaries](react-next/error-boundaries.md)
- [API Integration](react-next/api-integration.md)
- [Testing](react-next/testing.md)
- [Analytics & Tracking](react-next/analytics-tracking.md)
- [SEO & Metadata](react-next/seo-metadata.md)

---

## Governance, Versioning & Exceptions

- **Ownership:** this guideline is owned by `[Engineering leadership / architecture group]`, who review and approve changes.
- **Change process:** propose edits via **PR + ADR**; material changes are communicated to all active teams. Improvements discovered on a project flow back here rather than living only in that project.
- **Versioning:** the guideline carries a version and a dated changelog (below). Projects record which guideline version they adopted in their addendum.
- **Exceptions:** any deviation from a mandatory rule requires an ADR stating the rule, the reason, the scope, and a revisit trigger (date or milestone). Exceptions are time-boxed and reviewed — not permanent.

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| **0.1** | 2026-09-17 | Initial skeleton: folder structure and topic scaffolding for `shared/`, `frontend/`, and `react-next/` established. Content pending per file. |

*Guideline version: 0.1 — scaffold, content in progress*
