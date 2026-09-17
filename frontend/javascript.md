# JavaScript

**Read this first.** These are the baseline language and module conventions for any JavaScript running in the browser or a frontend build pipeline, independent of framework. They exist so code reads the same way across projects and across the TypeScript/plain-JS boundary. See [TypeScript Conventions](../shared/typescript-conventions.md) for typing rules — TypeScript is the default; this file covers what applies regardless.

**Applies to:** All frontend JavaScript/TypeScript source, including build scripts and config files.

---

## Mandatory

### Modules — ESM only
- Author and ship **ES modules** (`import`/`export`). No CommonJS (`require`/`module.exports`) in application source — only in Node-only tooling configs where the ecosystem still requires it (documented exception).
- One default export per file only when the file *is* the thing (a component, a single class). Prefer **named exports** for utilities — they survive refactors and rename tooling better than defaults.
- No wildcard re-export barrels (`export * from './x'`) at package/feature boundaries — they defeat tree-shaking and hide what a module actually exposes. Named re-exports are fine.

### No implicit globals
- Never assign to `window`/`globalThis` to pass data between modules. Use explicit imports, a typed context/store, or a dependency-injected client.
- Third-party scripts that must attach to `window` (analytics, payment SDKs) get a single typed accessor module — nothing else touches `window` directly.

### Async & error handling
- **`async`/`await` over raw `.then()` chains** for anything beyond a single call — chains hide error paths.
- Every `await` that can reject is inside a function with a `try/catch`, or the rejection is deliberately allowed to propagate to a caller that does catch it. No silently swallowed promises.
- Never leave a floating (un-awaited, uncaught) promise. If a call is intentionally fire-and-forget, mark it and handle its rejection explicitly:

```javascript
// Bad — unhandled rejection on failure
sendAnalyticsEvent(event);

// Good — explicit fire-and-forget with rejection handled
void sendAnalyticsEvent(event).catch((err) => logError('analytics', err));
```

- Use `Promise.allSettled` (not `Promise.all`) when independent operations should not fail as a group — e.g. loading three widgets where one failing shouldn't blank the page.
- Custom errors extend `Error`, carry a stable `name`, and preserve `cause` when wrapping:

```javascript
class ApiError extends Error {
  constructor(message, { status, cause } = {}) {
    super(message, { cause });
    this.name = 'ApiError';
    this.status = status;
  }
}
```

### Equality & type coercion
- **`===`/`!==` always.** `==` is banned by lint (see [Code Style & Linting](../shared/code-style-linting.md)) except the single idiomatic `value == null` check for "null or undefined."
- No implicit coercion in conditionals on values that could be `0`, `''`, or `NaN` — check the actual condition (`items.length === 0`, not `!items.length` when zero is a meaningful value worth naming).

### Immutability by default
- Prefer `const`; use `let` only when reassignment is real. Never `var`.
- Don't mutate function arguments or shared objects in place — return new values. Use spread/`structuredClone` for copies of plain data.

---

## Recommended (opt-in)

- **Functional composition over deep inheritance.** Class hierarchies beyond one level of extension are a smell in frontend code — prefer composed functions/hooks.
- **Feature detection over user-agent sniffing** when branching on browser capability (`'IntersectionObserver' in window`, not UA string parsing).
- Small utility libraries (date, deep-equal) are fine — check [Approved Libraries](../shared/approved-libraries.md) before adding one; often a 10-line native function is enough and avoids a dependency.

---

## Anti-Patterns (do not ship)

- `var`, `==`, or CommonJS `require` in application source.
- Mutating props/arguments, or mutating array/object state in place and expecting reactivity to notice.
- Catching an error only to `console.log` it and continue as if nothing happened — either handle it meaningfully or let it propagate.
- Deeply nested callback pyramids where `async`/`await` would flatten the logic.
- Attaching ad-hoc data to `window` to work around a module boundary.

## Quick Reference

```
✓ ESM only · named exports for utilities · no wildcard barrels
✓ async/await · try/catch on every awaited rejection path · no floating promises
✓ === always · const by default · no mutation of args/shared state
✓ Custom Error subclasses with cause preserved
✗ No var, ==, require() in app code
✗ No silent catch-and-ignore
✗ No window as a data bus between modules
```

---
*Section version: 0.1 — initial draft*
