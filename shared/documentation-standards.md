# Documentation Standards

**Read this first.** This governs what must be documented, where, and to what depth — code comments, README expectations, and component-library documentation. Undocumented shared code is a tax every future consumer pays.

**Applies to:** every repo adopting this guideline. Figma-to-component handoff documentation lives in [Design to Code](../frontend/design-to-code.md) — this file covers code-level and repo-level documentation.

---

## Mandatory

### JSDoc/TSDoc
Required on:
- Every exported function/hook/component in a **shared package** (consumed outside its own app).
- Any function whose behavior isn't obvious from its name and types alone (a non-obvious algorithm, a workaround for a specific bug, a side effect).

Not required on:
- Self-explanatory, well-typed internal functions (`camelCase` name + TS types already say what it does). TypeScript types replace redundant `@param`/`@returns` — don't restate the type in prose.

```typescript
/**
 * Debounces a value, only updating after `delayMs` of no changes.
 * Use for search-as-you-type inputs to avoid firing a request per keystroke.
 */
function useDebouncedValue<T>(value: T, delayMs: number): T {
  // ...
}
```

### README requirements (per repo/package)
Every repo's root `README.md` includes, at minimum:
- One-paragraph description of what the app/package does.
- Link to [Setup Checklist](setup-checklist.md) rather than duplicating setup steps.
- How to run tests, lint, and build locally.
- Link to the relevant guideline folders (this repo) and the project addendum (`docs/PROJECT_CONTEXT.md`).
- Any repo-specific deviation from this guideline, with a link to its ADR.

READMEs are updated **in the same PR** as the change that makes them stale — not as follow-up cleanup. A stale README is treated as a bug.

### Component documentation (design system / shared UI)
Every component published from a shared component library documents, at minimum:
- **Purpose** — what it's for, when to use it vs. a similar component.
- **Props table** — generated from TypeScript types where the tool supports it (e.g. Storybook autodocs), not hand-maintained separately.
- **Usage examples** covering the common variants and at least one edge case (e.g. long text, empty state, disabled).
- **Accessibility notes** — keyboard interaction, ARIA roles assumed, anything a consumer must do themselves (e.g. provide a label).

### Sample data & examples
- Code examples, fixtures, Storybook stories, and seed data use **obviously-fake data** — never a real user's name, email, phone number, or ID pulled from a real account for convenience. See [react/components-architecture.md](../frontend/react/components-architecture.md#privacy--data-boundaries-in-components) for the component-level version of this rule.
- Don't paste real production API responses (even "just for the shape") into docs or examples without scrubbing PII first — a scrubbed/synthetic fixture is the deliverable, not a raw payload.
- Example code doesn't wire in real analytics/tracking IDs or third-party keys — use placeholders (`YOUR_API_KEY`) so a copy-paste doesn't silently send data to production services.

### ADRs (Architecture Decision Records)
- Any hard-to-reverse decision (library choice, architectural pattern, deviation from a mandatory rule) gets an ADR in `docs/adr/`.
- ADR format: context, decision, consequences, and — for exceptions to a mandatory rule — a **revisit trigger** (date or milestone).
- ADRs are referenced from this guideline, not duplicated into it.

## Recommended (opt-in)

- **Storybook** (or equivalent) as the living documentation surface for a shared component library — see [Design to Code](../frontend/design-to-code.md) for the broader design-system workflow this plugs into.
- **Architecture diagrams** (e.g. C4-model) checked into `docs/architecture/` for repos complex enough that a README paragraph doesn't convey the shape — see [Project Architecture](project-architecture.md).
- **Automated docs generation** (TypeDoc) published to an internal site for large shared packages with many consumers.

## Anti-Patterns (do not ship)

- Comments that restate what the code obviously does (`// increment i` above `i++`).
- A README describing a setup process that no longer matches `package.json`/config — verify before merging a config change.
- Copy-pasting this guideline's content into a project repo instead of linking to it (creates drift — see root [README governance](../README.md#governance-versioning--exceptions)).
- Documenting a component's props by hand when the tooling can generate the table from types — hand-maintained tables go stale.
- Real user data, unscrubbed production payloads, or live API keys pasted into docs, examples, or fixtures.

## Quick Reference

```
✓ JSDoc/TSDoc on every shared-package export and non-obvious function
✓ README: what it does, setup link, how to test/lint/build, guideline links
✓ Docs updated in the same PR as the change that made them stale
✓ Shared components: purpose, generated props table, examples, a11y notes
✓ Hard-to-reverse decisions get an ADR with a revisit trigger for exceptions
✓ Examples/fixtures use fake data only; no real PII, no live keys
✗ No comments restating obvious code · no hand-duplicated prop tables · no stale setup docs
```

---
*Section version: 0.1 — initial draft*
