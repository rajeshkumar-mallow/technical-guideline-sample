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
*(Total TypeScript convention)*
- **`type` by default** — object shapes, unions, intersections, tuples, mapped/conditional types, component props, all of it. `type` and `interface` can express the same object shapes; defaulting to one keeps the codebase consistent.
- **`interface` only** when you specifically need declaration merging — augmenting a third-party or global type (`declare global`, extending a library's types) — or you're deliberately designing a public API surface that consumers should be able to extend via merging. Interfaces merge implicitly; that's a footgun everywhere else.
```typescript
// Object shape → type, not interface
type ButtonProps = {
  variant: 'primary' | 'secondary';
  onClick: () => void;
};

// Union → type
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

// Declaration merging is the one job interface does that type can't
interface Window {
  myGlobal: string; // augmenting a global — interface is correct here
}
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
- **Golden rule of generics** *(Total TypeScript convention)*: if a type parameter only appears once in the function signature, it doesn't need to be a generic — it's not relating multiple types to each other, so a generic buys nothing and just adds noise.
```typescript
// Bad — T is used once; it's not being related to anything else
function logAndReturn<T>(value: T): T {
  console.log(value);
  return value;
}

// Good — same behavior, no generic needed
function logAndReturn(value: unknown): unknown {
  console.log(value);
  return value;
}

// Good use of a generic — T ties the input and output together
function first<T>(items: T[]): T | undefined {
  return items[0];
}
```

### Null/undefined handling
- Prefer `undefined` over `null` for "absent" in application code; reserve `null` for cases mirroring an external API/DB contract that uses it explicitly.
- No non-null assertions (`!`) except immediately after a runtime check the compiler can't see (documented with a comment). Prefer optional chaining + nullish coalescing.

### No TS enums
*(Total TypeScript convention)*
`enum` and `const enum` are banned. They don't behave like other TypeScript types (numeric enums allow any number in, they generate runtime code non-`const` enums, and they don't erase cleanly), and they're one of the few TS features that isn't a strict superset of JavaScript. Use a union of string literals, or an `as const` object when you need the runtime values too.
```typescript
// Bad
enum Status {
  Idle,
  Loading,
  Success,
}

// Good — union of string literals
type Status = 'idle' | 'loading' | 'success';

// Good — when you need the values at runtime too
const Status = {
  Idle: 'idle',
  Loading: 'loading',
  Success: 'success',
} as const;
type Status = (typeof Status)[keyof typeof Status];
```

### Return types on exported functions
*(Total TypeScript convention)*
Annotate the return type explicitly on every exported/public function. Let TypeScript infer return types for unexported, internal helpers. Explicit return types on the public surface catch accidental type-widening at the source of the change instead of at every call site, and they make the function's contract readable without navigating to its implementation.
```typescript
// Exported — annotate
export function getUser(id: string): Promise<User> {
  return db.users.findById(id);
}

// Internal helper — inference is fine
function normalizeId(id: string) {
  return id.trim().toLowerCase();
}
```

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
- `enum` / `const enum` — use a union of string literals or an `as const` object instead.
- A generic type parameter used only once in a signature — inline the type instead.

## Quick Reference

```
✓ strict: true always · noUncheckedIndexedAccess on
✓ type by default · interface only for declaration merging
✓ unknown + narrowing instead of any · no bare any (eslint error)
✓ Discriminated unions over boolean-flag state
✓ Derive types from schema (Zod) — one source of truth
✓ Explicit return types on exported functions · infer internally
✗ No enum/const enum · no generic used only once · no @ts-ignore without a ticket
✗ No `as unknown as X` casts · no non-null `!` without a guard
```

**Further reading:** several conventions in this file (`type`-by-default, banning enums, the golden rule of generics, explicit return types on exported functions) follow [Matt Pocock](https://www.totaltypescript.com/)'s Total TypeScript material — worth a read for the reasoning behind each.

---
*Section version: 0.2 — adopted several Matt Pocock / Total TypeScript conventions: `type` over `interface` by default, banned enums, golden rule of generics, explicit return types on exported functions.*
