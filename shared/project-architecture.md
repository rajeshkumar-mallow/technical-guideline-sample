# Project Architecture

**Read this first.** Two decisions drive most long-term maintainability outcomes: how files are organized within an app, and when (if ever) to split beyond a single app. Both are covered here as defaults with explicit escalation triggers — not a menu of equally-valid options to pick from on day one.

**Applies to:** every project. React/Next-specific component-level architecture is in [react-next/components-architecture.md](../react-next/components-architecture.md).

---

## Mandatory

### Folder structure: feature-based, by default

Organize by **feature/domain**, not by technical type. A type-based structure (`components/`, `hooks/`, `utils/` at the root, all flat) works for small apps but stops scaling once feature count grows — you end up hunting across four top-level folders to understand one feature.

```
src/
  features/
    checkout/
      components/
      hooks/
      api.ts
      types.ts
      index.ts        # public exports only — everything else is internal to the feature
    user-profile/
      components/
      hooks/
      api.ts
      ...
  shared/              # truly cross-feature: design-system components, generic hooks, utils
    components/
    hooks/
    utils/
  app/                 # Next.js App Router tree, or top-level routing/composition for non-Next
```

- **A feature's internals are private.** Other features import only from a feature's `index.ts`, never reach into `features/checkout/components/SomeInternal.tsx` directly. This is what makes features independently understandable and refactorable — the same "can you understand a unit without reading its internals" test as any other module boundary.
- **`shared/` earns its place.** Something moves into `shared/` only after it's used by a second feature — don't pre-emptively extract "reusable" code that has one caller.
- Start flat (type-based) only for a genuinely tiny app (a handful of routes); migrate to feature-based **before** it hurts, not after — the earlier the switch, the cheaper it is.

### Architectural-style decision tree

Default to the simplest tier. Move up **only** when a concrete trigger is hit, and record the move as an ADR — this mirrors how the Rails guideline treats add-ons as opt-in, not default.

```
Tier 0 — Single app (DEFAULT)
  One Next.js/React app, feature-based structure above.
  Stays here for the overwhelming majority of projects, indefinitely.

  Trigger to move to Tier 1: the app needs a real backend beyond what
  Next.js API routes/Server Actions comfortably handle (heavy background
  jobs, non-JS services, a data layer several frontend apps share).
        │
        ▼
Tier 1 — Full-stack monolith vs. separate backend  (Next-specific fork)
  Option A: Next.js Server Actions / Route Handlers ARE the backend.
    Default choice — no separate service to deploy/version/auth against.
  Option B: Next.js is frontend-only, calls a separate API service.
    Choose this when: the API is also consumed by non-web clients
    (mobile, other services), or backend logic/team is genuinely
    independent of frontend release cadence.

  Trigger to move to Tier 2: one app's feature folders are growing
  tangled dependencies on each other despite the index.ts boundary —
  teams keep needing to reach into each other's internals.
        │
        ▼
Tier 2 — Modular monolith
  Still one deployable app, but feature boundaries become enforced,
  not just conventional: lint rule (e.g. eslint-plugin-boundaries) that
  fails a build if one feature imports another feature's internals.
  Consider extracting the most stable/shared features into internal
  packages in a monorepo (Turborepo/Nx) at this tier if build times
  or team ownership lines justify it.

  Trigger to move to Tier 3: multiple independent teams need to
  deploy to the same product surface on independent schedules, and
  a shared release train is now the bottleneck — not "the app got big."
        │
        ▼
Tier 3 — Micro-frontends
  Module Federation / single-spa / similar. Genuinely justified only
  by an organizational trigger (independent team deploy cadence),
  never by codebase size alone. Highest operational cost of any tier —
  requires an ADR with the specific trigger documented.
```

- **"The app got big" is never sufficient justification to skip a tier.** Tier 2 (enforced modular boundaries) solves most "big codebase" pain without the runtime/deploy complexity of Tier 3.
- Record the current tier and the ADR that justified any move past Tier 0/1A in the project addendum.

---

## Recommended (opt-in)

- **Monorepo tooling (Turborepo/Nx/pnpm workspaces)** once a project has more than one deployable app or more than one genuinely independent internal package — not for a single app, regardless of size.
- **Path aliases** (`@/features/checkout`) over long relative imports, configured once in `tsconfig.json`, to make the feature-based structure easier to navigate.

---

## Anti-Patterns (do not ship)

- Flat type-based structure (`components/`, `hooks/`, `utils/`) past the point where feature count makes it hard to navigate.
- Features reaching into each other's internals instead of importing through `index.ts`.
- Extracting to `shared/` on first write, "in case it's reused later."
- Adopting micro-frontends or a separate backend service because the codebase "feels big," with no specific organizational trigger.
- Skipping straight to Tier 3 for a project with one team and one release cadence.

---

## Quick Reference

```
✓ Feature-based folders by default · index.ts is the only public surface of a feature
✓ shared/ only after a second consumer exists
✓ Default to Tier 0 (single app) · escalate only on a concrete trigger, each move gets an ADR
✓ Next: Server Actions/Route Handlers as backend by default; separate API only when justified
✗ No architecture change justified by "it got big" alone
✗ No reaching into another feature's internals
```

---
*Section version: 0.1 — initial draft*
