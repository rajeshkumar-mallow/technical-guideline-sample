# Versioning & Release Process

**Read this first.** This governs how shared packages are versioned and released, and — separately — how the *guideline itself* and the framework versions it assumes stay current. Both are "versioning," but they're different concerns: one is about your code, the other is about not silently drifting from what's actually stable in the ecosystem.

**Applies to:** any repo publishing a shared package (component library, utility package, design tokens), and every repo tracking a Next.js/React baseline.

---

## Mandatory

### Semantic versioning for shared packages
- Every published internal package (component library, shared utils, design tokens) follows **semver**: `MAJOR.MINOR.PATCH`.
  - **MAJOR** — breaking change to a public API (prop removed/renamed, changed default behavior).
  - **MINOR** — new functionality, backward compatible.
  - **PATCH** — bug fix, no API change.
- Breaking changes are never shipped silently in a minor/patch — consumers must be able to trust the version number without reading the diff.

### Changesets
- Any PR that changes a published package's public behavior includes a **changeset** (via [Changesets](https://github.com/changesets/changesets) or equivalent) describing the change and its semver bump.
- The changeset's description becomes the changelog entry — write it for a *consumer*, not for the PR reviewer ("Fixed focus trap not releasing on `Escape`" not "fix bug in modal.tsx").

### Changelogs
- Every published package maintains a `CHANGELOG.md` (auto-generated from changesets is fine), append-only, most-recent first.
- This guideline itself carries a version and changelog at the root [README.md](../README.md) — material changes to mandatory rules are logged there.

### Deprecation policy
- A public API is never removed in the same release it's deprecated. Mark deprecated (JSDoc `@deprecated` + runtime warning where feasible in dev), give consumers at least one minor version cycle, then remove in the next major.

### Version currency review (framework/library baseline)
Separate from package releases: the **assumed framework baseline** (Next.js/React versions stated in [next/README.md](../frontend/next/README.md)) must not silently go stale.
- On every Next.js/React **minor** release, review: any newly-stabilized feature that was previously marked experimental in [next/rendering-strategies.md](../frontend/next/rendering-strategies.md)? Update that file's stable/experimental table.
- On every **major** release (or an approaching EOL of the current major), schedule an upgrade — track it the way the Rails baseline this guideline is modeled on tracks Ruby/Rails/PostgreSQL EOL: a living `docs/UPGRADE_PLAN.md` per project, reviewed on a cadence, not "when forced."
- Dependency currency more broadly: routine `npm outdated` / `pnpm outdated` review; no unmaintained/abandoned package added without an ADR and an exit plan — ties to [Approved Libraries](approved-libraries.md).

## Recommended (opt-in)

- **Automated release PRs** (Changesets' release workflow, or Release Please) that batch pending changesets into a version bump + changelog PR, merged to trigger publish.
- **Canary/prerelease channels** (`next` npm dist-tag) for a shared component library so consuming apps can opt into testing an upcoming breaking change before it's default.
- **Renovate/Dependabot** with grouped, scheduled PRs for dependency updates rather than ad hoc manual bumps, reducing review noise while keeping currency.

## Anti-Patterns (do not ship)

- Breaking a public prop/API in a patch or minor release.
- A shared package with no changelog — consumers forced to diff source to know what changed.
- Letting the stated framework baseline in `next/README.md` drift silently out of date while the actual project has moved on (or vice versa).
- Treating "experimental" framework features as production-stable without an ADR just because they work in dev.
- Pinning dependencies indefinitely to avoid the upgrade conversation instead of scheduling it.

## Quick Reference

```
✓ Semver for every published package; changeset per PR that changes public behavior
✓ Changelogs are consumer-facing, append-only, most-recent-first
✓ Deprecate ≥1 minor cycle before removing a public API
✓ Review stable-vs-experimental framework features on every minor release
✓ Track upgrades via docs/UPGRADE_PLAN.md — reviewed on a cadence, not when forced
✗ No breaking changes in patch/minor · no silently stale framework baseline · no indefinite dependency pinning
```

---
*Section version: 0.1 — initial draft*
