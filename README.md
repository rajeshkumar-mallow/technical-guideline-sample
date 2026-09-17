# Frontend Technical Guideline

**Read this first.** This is our standing technical standard for frontend and JavaScript/TypeScript work, with React/Next.js as the first framework filled in. Follow it during design, implementation, review, and local testing. It is not tied to any single project — projects adopt it as-is and record their specifics in a short project addendum.

**Status:** A living standard, maintained centrally and versioned. **Projects do not fork or rewrite it per engagement** — they follow the mandatory sections and record project-specific decisions in their own context doc. Propose changes to the guideline itself via PR + ADR (Architecture Decision Record).

**Scope note:** this guideline covers the **frontend** side only. It intentionally says nothing about backend language/framework choice (Rails, Python, PHP, Node, etc.) — `shared/` is written so it holds regardless of what the backend is.

**Structure:**

- **`shared/`** — truly language-agnostic rules: true whether the backend is Rails, Python, PHP, or Node, and whatever the frontend framework. Git workflow, testing philosophy, setup, approved libraries, environment/deployment, project architecture, documentation, versioning, feature flags, privacy.
- **`js/`** — JavaScript/TypeScript language- and tooling-level rules that apply to **any JS runtime**, browser or Node: code style/linting, TypeScript conventions, language conventions, testing tooling. Builds on `shared/`.
- **`frontend/`** — browser/DOM-specific, framework-agnostic rules: HTML/CSS, styling foundations, accessibility, performance, security, auth, PWA, i18n, forms, and more. Builds on `shared/` and `js/`.
- **`frontend/react/`** — React fundamentals (components, hooks, state management, forms, error boundaries, testing) that hold **regardless of meta-framework** — standalone (Vite + a router) or under Next.js.
- **`frontend/next/`** — Next.js (App Router)-specific rules that build on `frontend/react/`: routing, rendering strategies, Server Actions, middleware, and more.
- **`frontend/<other-framework>/`** — `frontend/preact/`, `frontend/stimulus/`, `frontend/vue/` are reserved placeholders, to be filled in as other teams' stacks require. Preact is React-API-compatible (via `preact/compat`) and would mostly reuse `frontend/react/`; Vue and Stimulus are not part of the React family and would not.
- **`backend/`** — reserved for a future Node.js backend guideline. Out of scope for now.

A Next.js project follows `shared/` + `js/` + `frontend/` + `frontend/react/` + `frontend/next/`. A standalone React project follows `shared/` + `js/` + `frontend/` + `frontend/react/` alone. A project on a non-React frontend framework follows `shared/` + `js/` + `frontend/` + that framework's subfolder (once written).

---

## How to Use This Guideline

- The rules in `shared/`, `js/`, and `frontend/` (plus the relevant framework subfolder) apply to every project that adopts this guideline, unless an ADR documents a justified exception.
- **Do not edit these documents per project.** Each project keeps a short **project addendum** (e.g. `docs/PROJECT_CONTEXT.md`) recording what varies — stack versions, enabled add-ons, out-of-scope items, open TBCs.
- Treat this as a living standard: improvements discovered on a project flow back here via PR + ADR, so every future project benefits.
- Version baselines (e.g. which Next.js/React version is assumed) are stated in `frontend/next/README.md`. Fast-moving framework features are explicitly marked **stable** vs. **experimental** — see `frontend/next/rendering-strategies.md` — and experimental features require an ADR before adoption in production.

---

## Table of Contents

### shared/ — language-agnostic
- [Git & PR Workflow](shared/git-pr-workflow.md)
- [Testing Philosophy](shared/testing-philosophy.md)
- [Setup Checklist](shared/setup-checklist.md)
- [Approved Libraries](shared/approved-libraries.md)
- [Environment & Configuration](shared/environment-config.md)
- [Deployment](shared/deployment.md)
- [Project Architecture](shared/project-architecture.md)
- [Documentation Standards](shared/documentation-standards.md)
- [Versioning & Release Process](shared/versioning-release-process.md)
- [Feature Flags](shared/feature-flags.md)
- [Privacy & Data Compliance](shared/privacy-compliance.md)

### js/ — any JS runtime
- [Code Style & Linting](js/code-style-linting.md)
- [TypeScript Conventions](js/typescript-conventions.md)
- [JavaScript](js/javascript.md)
- [Testing Tooling](js/testing-tooling.md)

### frontend/ — browser-specific
- [HTML & CSS](frontend/html-css.md)
- [Styling Foundations](frontend/styling-foundations.md)
- [Design to Code](frontend/design-to-code.md)
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
- [Analytics & Tracking](frontend/analytics-tracking.md)
- [SEO & Metadata](frontend/seo-metadata.md)
- [API Resilience](frontend/api-resilience.md)
- [Browser & Device Support Matrix](frontend/browser-support-matrix.md)

### frontend/react/ — active, framework-agnostic
- [Components & Architecture](frontend/react/components-architecture.md)
- [State Management](frontend/react/state-management.md)
- [Hooks](frontend/react/hooks.md)
- [Forms & Validation](frontend/react/forms-validation.md)
- [Error Boundaries](frontend/react/error-boundaries.md)
- [Testing](frontend/react/testing.md)

### frontend/next/ — active, builds on frontend/react/
- [Routing & Data Fetching](frontend/next/routing-data-fetching.md)
- [Rendering Strategies](frontend/next/rendering-strategies.md)
- [Performance](frontend/next/performance.md)
- [Security](frontend/next/security.md)
- [Authentication](frontend/next/auth.md)
- [Internationalization](frontend/next/i18n.md)
- [Forms & Validation](frontend/next/forms-validation.md)
- [Error Boundaries](frontend/next/error-boundaries.md)
- [API Integration](frontend/next/api-integration.md)
- [Testing](frontend/next/testing.md)
- [Analytics & Tracking](frontend/next/analytics-tracking.md)
- [SEO & Metadata](frontend/next/seo-metadata.md)

### Reserved (not yet written)
- [`frontend/preact/`](frontend/preact/README.md)
- [`frontend/stimulus/`](frontend/stimulus/README.md)
- [`frontend/vue/`](frontend/vue/README.md)
- [`backend/`](backend/README.md)

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
| **0.4** | 2026-09-17 | Moved `js/backend/` to top-level `backend/`, a sibling of `shared/`/`js/`/`frontend/` rather than nested under `js/` — matches the pattern that backend language/framework is independent of the JS/TS layer, and leaves room for non-Node backend guidelines later. |
| **0.3** | 2026-09-17 | Split `frontend/react-next/` into `frontend/react/` (framework-agnostic React fundamentals — components, state, hooks, forms, error boundaries, testing) and `frontend/next/` (Next.js App Router-specific rules that build on `frontend/react/`), so a standalone React project (no Next.js) can adopt `frontend/react/` alone, and `frontend/preact/` can reuse it later. |
| **0.2** | 2026-09-17 | Restructured to separate language-agnostic (`shared/`), JS/TS cross-runtime (`js/`), and browser-specific (`frontend/`) concerns; moved `react-next/` under `frontend/` as a framework subfolder; added reserved placeholders (`frontend/preact/`, `frontend/stimulus/`, `frontend/vue/`, `js/backend/`) for future stacks. |
| **0.1** | 2026-09-17 | Initial skeleton: folder structure and topic scaffolding for `shared/`, `frontend/`, and `react-next/` established. Content pending per file. |

*Guideline version: 0.4*
