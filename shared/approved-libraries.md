# Approved Libraries

**Read this first.** Uncontrolled dependency sprawl is the fastest way to inflate bundle size, create supply-chain risk, and leave the codebase with three different libraries doing the same job. This file is the pre-approved list plus the process for adding anything not on it.

**Applies to:** every project; React/Next-specific packages are cross-referenced from [`react-next/`](../react-next/README.md) rather than duplicated here.

---

## Mandatory

### Pre-approved core (use these by default — no justification needed)

| Category | Package | Notes |
|---|---|---|
| Language | `typescript` | See [TypeScript Conventions](typescript-conventions.md) |
| Linting/format | `eslint`, `prettier` (or `biome`) | See [Code Style & Linting](code-style-linting.md) |
| Unit/component testing | `vitest`, `@testing-library/react` | See [Testing Tooling](../frontend/testing-tooling.md) |
| E2E testing | `@playwright/test` | Preferred over Cypress for new projects — see [Testing Tooling](../frontend/testing-tooling.md) |
| Dates | `date-fns` (or native `Intl` / `Temporal` where sufficient) | Never `moment` (unmaintained) |
| Forms | `react-hook-form` + `zod` | See [Forms & Validation](../react-next/forms-validation.md) |
| Server state / data fetching | `@tanstack/react-query` (or framework-native fetching in Next App Router) | See [State Management](../react-next/state-management.md) |
| Client global state | `zustand` | Only when Context is insufficient — see [State Management](../react-next/state-management.md) |
| HTTP client | native `fetch` | Wrap with retry/timeout per [API Resilience](../frontend/api-resilience.md); avoid `axios` unless a specific interceptor need justifies it |
| Utility | `clsx` / `tailwind-merge` | Class-name composition only — no general-purpose utility-belt libraries (no `lodash` for one function) |
| Icons | project's chosen icon set (record in project addendum) | One icon library per project, not several |
| Error tracking | Sentry SDK | See [Error Monitoring](../frontend/error-monitoring.md) |
| Animation | native CSS transitions / `@keyframes` first; `motion` (Framer Motion) if complex orchestration is genuinely needed | See [Accessibility](../frontend/accessibility.md) for `prefers-reduced-motion` |

### Adding a new dependency — process

A new package is not "just installed." Before adding one:

1. **Check this list and the project's existing `package.json` first** — the need is very often already met.
2. **Justify it in the PR description**: what problem it solves, why an existing approved package can't, and its weekly download count / last-publish date / maintenance signal.
3. **Check the bundle-size cost** for anything client-shipped:
   ```bash
   npx bundle-phobia <package-name>
   # or, post-install:
   pnpm build && pnpm analyze   # project's bundle-analyzer script
   ```
   A single new dependency adding **> 10KB gzipped** to a client bundle needs explicit sign-off in the PR, not a silent merge.
4. **Run the supply-chain check** — `pnpm audit` (or `npm audit`) must show no unresolved high/critical vulnerabilities introduced by the new package.
5. **Prefer packages that are**: actively maintained (commit in the last 6 months), have TypeScript types (native or `@types/*`), and have no known unpatched CVEs.
6. **Record it via ADR** if the package materially changes an architectural pattern (e.g. adopting a new state-management paradigm, swapping the HTTP client project-wide) — a leaf utility does not need an ADR, a paradigm shift does.

---

## Recommended (opt-in)

- **Automated dependency updates** (Renovate/Dependabot) with grouped, scheduled PRs rather than ad-hoc manual bumps.
- **License scanning** (`license-checker`) once a project has legal/compliance obligations — pairs with [Privacy & Data Compliance](privacy-compliance.md).
- **Bundle-size budgets enforced in CI** (`size-limit` / Next's built-in bundle analyzer thresholds) once a project has shipped its first performance regression from an uncontrolled dependency.

---

## Anti-Patterns (do not ship)

- Adding a dependency for something the standard library or an already-approved package already does (`lodash.debounce` when one 5-line debounce util suffices).
- Two libraries solving the same problem in one codebase (e.g. both `axios` and `fetch` wrappers, both `moment` and `date-fns`).
- Installing a package with zero recent commits or open critical CVEs "because it works."
- Silently absorbing a large transitive dependency without checking its bundle-size impact.
- Pinning to `latest`/`*` in `package.json` instead of a resolvable semver range matched by the lockfile.

---

## Quick Reference

```
✓ Use the pre-approved list by default · check it before adding anything new
✓ New dependency: justify in PR + check bundle-phobia + `pnpm audit` clean
✓ >10KB gzipped addition needs explicit PR sign-off
✓ Paradigm-shifting dependency changes get an ADR
✗ No duplicate libraries solving the same problem
✗ No unmaintained/CVE-flagged packages · no `*`/`latest` version pins
```

---
*Section version: 0.1 — initial draft*
