# Code Style & Linting

**Read this first.** This governs naming, formatting, and the lint/format toolchain shared by every frontend, React, and Next.js repo. Consistent style removes a whole category of PR review noise and diff churn.

**Applies to:** all JavaScript/TypeScript source in every repo that adopts this guideline. Component-specific conventions live in [react-next/components-architecture.md](../react-next/components-architecture.md); type-level rules live in [TypeScript Conventions](typescript-conventions.md).

---

## Mandatory

### Toolchain
- **ESLint** for linting, **Prettier** for formatting — never hand-roll formatting rules in ESLint that Prettier already owns (`eslint-config-prettier` disables the overlap).
- **EditorConfig** (`.editorconfig`) for cross-editor whitespace/EOL/charset baseline.
- Format-on-save and a **pre-commit hook** (`lint-staged` + `husky` or equivalent) running Prettier + ESLint `--fix` on staged files.
- CI runs `eslint .` and `prettier --check .` as required checks — a PR cannot merge with lint errors, only pre-existing warnings tracked as debt ([Technical Debt] convention TBD).

### Base ESLint config
```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react-hooks/recommended",
    "plugin:jsx-a11y/recommended",
    "prettier"
  ],
  "rules": {
    "no-console": ["warn", { "allow": ["warn", "error"] }],
    "no-unused-vars": "off",
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "@typescript-eslint/no-explicit-any": "error"
  }
}
```
`jsx-a11y` is mandatory wherever JSX is used — it is the first line of defense referenced in [Accessibility](../frontend/accessibility.md).

### Naming conventions
| Kind | Convention | Example |
|---|---|---|
| Variables, functions | `camelCase` | `getUserProfile` |
| React components, classes, types, interfaces | `PascalCase` | `UserAvatar`, `ApiResponse` |
| Constants (true constants, not just `const` bindings) | `SCREAMING_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Files — components | `PascalCase.tsx` | `UserAvatar.tsx` |
| Files — everything else | `kebab-case.ts` | `format-currency.ts` |
| Boolean variables/props | `is`/`has`/`can`/`should` prefix | `isLoading`, `hasError` |
| Event handler props | `on` prefix; handler implementations `handle` prefix | `onSubmit` prop, `handleSubmit` function |
| Custom hooks | `use` prefix | `useDebouncedValue` |

### Formatting baseline (Prettier)
```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```
Do not bikeshed these in individual PRs — changing the shared config is itself a PR against this guideline.

### Import order
Enforced via `eslint-plugin-import` or `simple-import-sort`:
1. Node/external packages
2. Internal absolute imports (`@/...`)
3. Relative imports (`../`, `./`)
4. Styles

## Recommended (opt-in)

- **`eslint-plugin-unicorn`** for additional correctness/consistency rules beyond the base set.
- **Stylelint** for CSS/SCSS files if the project uses stylesheets rather than a utility-first/CSS-in-JS approach — see [Styling Foundations](../frontend/styling-foundations.md).
- **Commitlint** integration to enforce [Git & PR Workflow](git-pr-workflow.md) commit conventions at commit time, not just in review.
- Editor-level ESLint/Prettier extensions with workspace settings checked in (`.vscode/settings.json`) so defaults are consistent without per-developer setup.

## Anti-Patterns (do not ship)

- Disabling a lint rule inline (`// eslint-disable-next-line`) without a comment explaining why.
- Committing with `--no-verify` to skip pre-commit hooks.
- Project-specific ESLint config forks that diverge from the shared base without an ADR.
- Mixing tabs and spaces, or ignoring `.editorconfig` in an IDE.
- `any`-typed escape hatches to silence lint errors instead of fixing the type — see [TypeScript Conventions](typescript-conventions.md).

## Quick Reference

```
✓ ESLint + Prettier + EditorConfig, pre-commit hook via lint-staged
✓ jsx-a11y mandatory wherever JSX is used
✓ camelCase vars/fns · PascalCase components/types · kebab-case non-component files
✓ Shared Prettier config — don't bikeshed per PR
✗ No inline eslint-disable without a reason comment · no --no-verify commits
```

---
*Section version: 0.1 — initial draft*
