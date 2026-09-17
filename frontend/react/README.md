# React Guidelines

Framework-agnostic React rules — the same whether the app runs standalone (e.g. Vite + React Router) or inside a meta-framework. Builds on [`../../shared/`](../../shared/README.md), [`../../js/`](../../js/README.md), and [`../`](../README.md).

For Next.js, this folder is the **base layer** — [`../next/`](../next/README.md) builds on it and adds App Router/RSC-specific rules (routing, rendering strategies, Server Actions, middleware). A standalone React project follows this folder alone, plus a router/build choice of its own (not yet written here — add when a project needs it).

| File | Covers |
|------|--------|
| [Components & Architecture](components-architecture.md) | Folder conventions, composition, props/typing, privacy boundaries |
| [State Management](state-management.md) | Local vs. global state, context, server state |
| [Hooks](hooks.md) | Custom hook conventions, rules of hooks |
| [Forms & Validation](forms-validation.md) | react-hook-form + zod |
| [Error Boundaries](error-boundaries.md) | The React error-boundary mechanism |
| [Testing](testing.md) | React Testing Library, component/hook testing |

Back to [`frontend/`](../README.md) · [root guideline](../../README.md).
