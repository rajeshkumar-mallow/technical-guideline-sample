# TypeScript Conventions

**Read this first.** This governs how types are written across every frontend, React, and Next.js repo. TypeScript is only as useful as its strictness and consistency — a codebase full of `any` provides false confidence.

**Applies to:** all `.ts`/`.tsx` source. React-specific typing (props, generics on components, hooks) is covered here; component structure itself is in [react/components-architecture.md](../frontend/react/components-architecture.md).

---

## Mandatory

### Compiler configuration
`strict: true` is non-negotiable baseline. No project ships with it disabled.
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true
  }
}
```

### `interface` vs. `type`
- **`interface`** for object shapes that may be extended or implemented (component props, public API contracts). Interfaces merge; use that only intentionally (e.g. augmenting third-party types).
- **`type`** for unions, intersections, tuples, mapped/conditional types, and anything that isn't a plain extendable object shape.
```typescript
// Object shape, extendable → interface
interface ButtonProps {
  variant: 'primary' | 'secondary';
  onClick: () => void;
}

// Union → type
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

### Never use bare `any`
- `@typescript-eslint/no-explicit-any` is `error`, not `warn` (see [Code Style & Linting](code-style-linting.md)).
- When the type is genuinely unknown (e.g. parsing external JSON), use `unknown` and narrow it — never `any`.
```typescript
// Bad
function parse(json: string): any {
  return JSON.parse(json);
}

// Good
function parse(json: string): unknown {
  return JSON.parse(json);
}

function isUser(value: unknown): value is User {
  return typeof value === 'object' && value !== null && 'id' in value;
}
```
- The only sanctioned `any` is a narrowly-scoped, commented escape hatch at a third-party boundary with no types available, immediately wrapped in a typed function.

### Generics
- Name generics meaningfully when there's more than one (`TInput`, `TOutput`), not blindly `T, U, V` past the first.
- Constrain generics rather than leaving them unbounded when the function only works on a shape:
```typescript
function getField<T extends Record<string, unknown>, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

### Null/undefined handling
- Prefer `undefined` over `null` for "absent" in application code; reserve `null` for cases mirroring an external API/DB contract that uses it explicitly.
- No non-null assertions (`!`) except immediately after a runtime check the compiler can't see (documented with a comment). Prefer optional chaining + nullish coalescing.

### Discriminated unions over boolean flags
```typescript
// Bad — invalid states are representable (loading: true, error: set)
interface State {
  loading: boolean;
  error?: Error;
  data?: User;
}

// Good — invalid states are unrepresentable
type State =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'error'; error: Error }
  | { status: 'success'; data: User };
```

### Utility types
Use built-in utility types (`Partial`, `Pick`, `Omit`, `Record`, `ReturnType`) instead of hand-duplicating a shape. Derive types from a single source of truth (e.g. a Zod schema — see [Forms & Validation](../frontend/forms-validation.md)) rather than maintaining a type and a runtime validator in parallel.

## Recommended (opt-in)

- **`ts-reset`** or similar to tighten default lib types (e.g. `.json()` returning `unknown` instead of `any`).
- **Branded/nominal types** for primitives that shouldn't be interchangeable (`UserId` vs. `OrderId` both being `string`), when a codebase has had bugs from mixing them.
- **`satisfies`** operator over type annotations when you want literal-type inference preserved while still checking shape conformance.

## Anti-Patterns (do not ship)

- `// @ts-ignore` or `// @ts-expect-error` without a linked ticket explaining why the error is expected and when it'll be resolved.
- Casting through `as unknown as X` to force an incompatible type — a strong signal the types are wrong, not the cast.
- Re-declaring a shape that a schema library (Zod) already infers — two sources of truth drift.
- Optional properties (`field?:`) used to mean "can be any of several states" instead of a proper discriminated union.

## Quick Reference

```
✓ strict: true always · noUncheckedIndexedAccess on
✓ interface for extendable object shapes · type for unions/intersections
✓ unknown + narrowing instead of any · no bare any (eslint error)
✓ Discriminated unions over boolean-flag state
✓ Derive types from schema (Zod) — one source of truth
✗ No @ts-ignore without a ticket · no `as unknown as X` casts · no non-null `!` without a guard
```

---
*Section version: 0.1 — initial draft*
