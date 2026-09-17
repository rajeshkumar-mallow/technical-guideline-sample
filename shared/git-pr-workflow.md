# Git & PR Workflow

**Read this first.** This governs how branches, commits, and pull requests move through the repo. Consistent workflow keeps history readable, makes reverts safe, and keeps CI signal trustworthy.

**Applies to:** every frontend, React, and Next.js repo that adopts this guideline.

---

## Mandatory

### Branching model
- **Trunk-based**: `main` is always deployable. Short-lived feature branches, merged frequently (aim for < 3 days of age).
- Branch naming: `<type>/<ticket-id>-<short-slug>`, e.g. `feat/APP-421-checkout-summary`, `fix/APP-503-safari-date-picker`.
- Types: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `perf` — matches commit types below.
- No direct commits to `main`. No long-lived `develop`/`staging` branches unless [Deployment](deployment.md) defines an explicit release-train model.

### Commit conventions
- **Conventional Commits** format, enforced by commit-lint in CI:
  ```
  <type>(<scope>): <summary>

  <body — the why, not the what>

  <footer — ticket ref, breaking-change note>
  ```
  ```
  fix(checkout): prevent double-submit on slow networks

  Submit button stayed enabled during the in-flight request, so a
  slow connection let users fire duplicate orders.

  Closes APP-503
  ```
- One logical change per commit. Squash-merge feature branches into a single commit on `main` unless the PR is a deliberate multi-commit series reviewed as such.
- No `WIP`, `fixup`, or `asdf`-style commits on `main` — clean those up before merge (`git rebase -i`).

### PR requirements
- **Every PR** links a ticket, states what changed and why, and includes a testing note (what you ran, what you couldn't test).
- **Minimum 1 approval** before merge; 2 for changes touching auth, payments, or shared/`shared/`-level conventions.
- **CI must be green**: lint ([Code Style & Linting](code-style-linting.md)), type-check ([TypeScript Conventions](typescript-conventions.md)), tests ([Testing Philosophy](testing-philosophy.md)), build.
- PRs stay small: target < 400 changed lines (excluding generated/lockfiles). Split larger work into a stacked series.
- **No self-merge** except for docs-only or config-only changes explicitly marked low-risk.

### PR review checklist (author fills before requesting review)
| Check | |
|---|---|
| Linked to a ticket | ☐ |
| Tests added/updated for behavior change | ☐ |
| No `console.log` / debug code left in | ☐ |
| Accessibility considered for UI changes ([Accessibility](../frontend/accessibility.md)) | ☐ |
| No secrets or hardcoded environment values ([Environment & Configuration](environment-config.md)) | ☐ |
| Screenshots/recording attached for visual changes | ☐ |

### Merge strategy
- **Squash and merge** is the default. Merge commits only for release/train branches if [Deployment](deployment.md) uses one.
- Delete the branch on merge.

## Recommended (opt-in)

- **Stacked PRs** (e.g. via Graphite) for large features that must land incrementally — record the convention in an ADR once a team adopts it, so it doesn't become an unstated norm.
- **Auto-labeling / size-labeling bots** to flag oversized PRs before human review.
- **Changesets** (see [Versioning & Release Process](versioning-release-process.md)) for repos that publish shared packages — each PR touching a published package includes a changeset file.
- **Draft PRs** for early feedback on approach before implementation is complete.

## Anti-Patterns (do not ship)

- Force-pushing to `main` or a shared branch other contributors have based work on.
- PRs with no description beyond the auto-generated diff summary.
- Merging with failing or skipped CI checks "to unblock," without a follow-up ticket.
- Rewriting history on a branch already merged.
- Approving your own PR, or rubber-stamp approvals with no actual read of the diff.

## Quick Reference

```
✓ Trunk-based, short-lived branches, main always deployable
✓ Conventional Commits, squash-merge to main
✓ Every PR: ticket link, why, testing note, ≥1 approval (2 for auth/payments/shared)
✓ CI green (lint, type-check, test, build) before merge
✓ PRs < 400 lines; split larger work
✗ No force-push to main · no self-merge (except docs/config) · no WIP commits on main
```

---
*Section version: 0.1 — initial draft*
