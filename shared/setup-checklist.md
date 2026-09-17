# Setup Checklist

**Read this first.** This is the exact sequence a new engineer follows to get a working local environment before opening their first PR. Skipping steps here is the single biggest source of "works on my machine" bugs.

**Applies to:** every project adopting this guideline (frontend-only and React/Next alike).

---

## Mandatory

### Before you write any code

1. **Install the pinned runtime.** Use the version manager the project specifies (`.nvmrc` / `.node-version` / Volta `"volta"` block in `package.json`) — never a globally installed Node. Run `nvm use` (or `volta install`) before anything else.
2. **Install the pinned package manager.** The project's `package.json#packageManager` field (Corepack-enabled) is the source of truth — run `corepack enable` once, then let it resolve the exact pnpm/yarn/npm version. Never mix package managers in one repo (no `package-lock.json` next to `pnpm-lock.yaml`).
3. **Install dependencies with the frozen lockfile**, never a bare install that may resolve fresh versions:
   ```bash
   pnpm install --frozen-lockfile
   # or: npm ci   |   yarn install --immutable
   ```
4. **Copy the env template and fill it in.** Every project ships `.env.example` with every required variable and a safe placeholder (see [Environment & Configuration](environment-config.md)). Never copy a teammate's real `.env`.
   ```bash
   cp .env.example .env.local
   ```
5. **Install editor tooling.** ESLint, Prettier (or Biome), and the TypeScript extension must be active in-editor, pointed at the project's config — not global defaults. See [Code Style & Linting](../js/code-style-linting.md).
6. **Install git hooks.** `pnpm prepare` (Husky/`simple-git-hooks`) wires up pre-commit lint/format and commit-msg validation. Do not bypass with `--no-verify` (see [Git & PR Workflow](git-pr-workflow.md)).
7. **Start the dev server** using the project script, not a raw framework command someone remembers:
   ```bash
   pnpm dev
   ```
8. **Run the full quality gate locally at least once** before your first commit, so failures are yours, not CI's:
   ```bash
   pnpm lint && pnpm typecheck && pnpm test && pnpm build
   ```
9. **Read open ADRs** in `docs/adr/` — they capture decisions and exceptions that aren't obvious from the code.
10. **Read the project addendum** (`docs/PROJECT_CONTEXT.md`) for what varies from this guideline's baseline (stack versions, enabled optional sections, out-of-scope items).

### First-PR checklist

- [ ] Branch created per [Git & PR Workflow](git-pr-workflow.md) naming convention.
- [ ] `pnpm lint && pnpm typecheck && pnpm test` pass locally.
- [ ] No secrets committed — check `git diff` before pushing, not after.
- [ ] New dependency, if any, is either pre-approved (see [Approved Libraries](approved-libraries.md)) or justified in the PR description.
- [ ] Screenshots/recording attached for any visual change.
- [ ] PR description explains *why*, not just *what*.

---

## Recommended (opt-in)

- **Devcontainer / Docker Compose** for full environment parity (matches CI exactly) — adopt once "works on my machine" issues recur more than once a quarter; record the choice in an ADR since it adds a maintenance surface.
- **`mise`/`asdf`** instead of per-tool version managers, when a project juggles more than just Node (e.g. also Ruby/Python for tooling scripts).
- **Onboarding script** (`bin/setup`) that runs steps 1–8 automatically — worth it once a team has hired its third new engineer.

---

## Anti-Patterns (do not ship)

- Installing dependencies without a frozen lockfile ("just run `npm install`").
- Global framework CLIs (`npm i -g next`) instead of the project's local, versioned one.
- Committing a filled-in `.env` instead of leaving `.env.example` as the template.
- Skipping the local quality gate and letting CI be the first ESLint/TypeScript run.
- `git commit --no-verify` to skip a failing hook instead of fixing the failure.

---

## Quick Reference

```
✓ Pinned Node via .nvmrc/Volta · Corepack-managed package manager · frozen lockfile install
✓ .env.local from .env.example · never commit real secrets
✓ Editor linting/formatting/TS active · git hooks installed via `pnpm prepare`
✓ pnpm dev to start · full quality gate run locally before first commit
✓ Read docs/adr/ and docs/PROJECT_CONTEXT.md before first PR
✗ No global CLI installs · no bare `npm install` without lockfile · no --no-verify
```

---
*Section version: 0.1 — initial draft*
