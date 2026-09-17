# Tooling & Build

**Read this first.** A consistent toolchain across projects means a developer moving between them isn't relearning build config every time, and CI stays predictable. This file sets the default bundler and package manager; deviate only with an ADR.

**Applies to:** All frontend projects (any framework). Next.js projects use Next's built-in build pipeline — see [next/README.md](next/README.md) — but the package-manager and browser-target rules here still apply.

---

## Mandatory

### Bundler — Vite (default)
- **Vite** is the default bundler/dev-server for non-Next projects: fast native-ESM dev server, esbuild-powered pre-bundling, Rollup for production builds.
- Framework-specific meta-frameworks (Next.js) bring their own build pipeline — don't hand-roll a competing Vite config alongside Next.
- Any deviation (Webpack, Turbopack standalone, Parcel, esbuild-direct) requires an ADR stating the reason (e.g. a legacy plugin with no Vite equivalent).

### Package manager — pnpm (default)
- **pnpm** is the default package manager: content-addressable store (fast installs, low disk usage), strict dependency resolution (a package can't silently import an undeclared transitive dependency — catches phantom dependency bugs npm/yarn allow).
- **Lockfile is committed and is the source of truth.** CI installs with `--frozen-lockfile` (pnpm) — a build never silently re-resolves versions.
- One package manager per repo — never mix lockfiles (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` should never coexist).
- Monorepos use **pnpm workspaces**; see [Project Architecture](../shared/project-architecture.md) for when a monorepo is warranted at all.

### Build configuration conventions
- Build config lives in version-controlled files (`vite.config.ts`, `tsconfig.json`) — no build behavior toggled by undocumented local environment state.
- Environment-specific build output (dev/staging/prod) is driven by **build-time env vars** (see [Environment & Configuration](../shared/environment-config.md)), not by branching logic keyed on a hostname check in application code.
- Path aliases (`@/components`, `@/lib`) are defined once in `tsconfig.json` and mirrored in the bundler config — never diverge between the two, or editor autocomplete and the actual build disagree.
- Source maps: enabled in dev always; enabled in production builds but uploaded to the error-tracking provider and **not served publicly** (see [Error Monitoring](error-monitoring.md)).

### Browser targets
- The bundler's target/transpile list is driven by the project's stated [Browser & Device Support Matrix](browser-support-matrix.md) — not a default guess. Set `browserslist` (or Vite's `build.target`) explicitly.
- Ship modern JS to modern browsers by default (`type="module"` + `nomodule` fallback, or a differential-serving setup) rather than transpiling everything down to the lowest common denominator for every user.

---

## Recommended (opt-in)

- `vite-plugin-inspect` or the bundle analyzer during investigation of a budget regression (ties to [Performance](performance.md)).
- A `pnpm dlx` / `npx`-free workflow: scripts declared in `package.json` so onboarding never depends on globally installed CLIs beyond Node and pnpm itself.

---

## Anti-Patterns (do not ship)

- Committing more than one lockfile format.
- A build that behaves differently on a teammate's machine because config lives outside version control.
- Path aliases defined in the bundler but missing from `tsconfig.json` (or vice versa).
- Transpiling to ES5 for a project whose support matrix only requires evergreen browsers.
- Publicly served production source maps exposing unminified source.

## Quick Reference

```
✓ Vite default bundler (Next projects use Next's own pipeline)
✓ pnpm default package manager · lockfile committed · --frozen-lockfile in CI
✓ One lockfile format per repo · pnpm workspaces for monorepos
✓ Build config version-controlled · path aliases match between tsconfig and bundler
✓ Browser target driven by the support matrix, not guessed
✗ No mixed lockfiles · no undocumented local-only build behavior
✗ No public prod source maps
```

---
*Section version: 0.1 — initial draft*
