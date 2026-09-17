# Components & Architecture

**Read this first.** How components are organized and composed determines whether a codebase stays easy to change or turns into prop-drilling and copy-paste. This file covers React component conventions that hold **regardless of meta-framework** (Next.js, a standalone Vite app, or otherwise). Framework-specific concerns — like the Server/Client Component boundary, which only exists under React Server Components (currently a Next.js App Router feature) — live in [`../next/rendering-strategies.md`](../next/rendering-strategies.md).

**Applies to:** any React codebase.

---

## Mandatory

### Folder conventions
- **Colocate** feature/route-specific components next to the feature or route that owns them (e.g. a `_components/` or `components/` folder scoped to that feature) rather than in one flat global folder.
- **Shared/reusable** components live in a top-level `components/` (or `src/components/`), organized by **domain**, not by type (`components/billing/`, not `components/buttons/` + `components/forms/` split arbitrarily).
- One component per file; filename matches the component name (`invoice-table.tsx` exports `InvoiceTable`).

### Props & typing
- Every component's props are an explicit `interface` or `type`, never `any`, never untyped destructuring from an inferred object (see [`typescript-conventions.md`](../../js/typescript-conventions.md) for the broader TS baseline).
- Prefer **composition (`children`, render props, slots)** over boolean prop explosion (`variant`, `size` are fine; `showHeader`, `hideFooter`, `isCompactWithBorder` is a sign to split the component).

```tsx
interface InvoiceTableProps {
  invoices: Invoice[];
  onRowClick?: (id: string) => void;
}

export function InvoiceTable({ invoices, onRowClick }: InvoiceTableProps) {
  // ...
}
```

### Composition patterns
- **Compound components** for complex, multi-part UI where the pieces share implicit state — adopt when a feature has genuine internal composition needs, not as a default pattern:

```tsx
<Tabs defaultValue="account">
  <Tabs.List>
    <Tabs.Trigger value="account">Account</Tabs.Trigger>
    <Tabs.Trigger value="billing">Billing</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Content value="account">...</Tabs.Content>
  <Tabs.Content value="billing">...</Tabs.Content>
</Tabs>
```

- **React 19: prefer `use()` over `useContext()`** for reading context — `use()` can be called conditionally (after an early return, inside an `if`), unlike `useContext()`, and also unwraps promises when reading async data:

```tsx
// Preferred (React 19)
function Avatar() {
  const theme = use(ThemeContext);
  // ...
}

// Avoid — useContext() cannot be called conditionally
function Avatar() {
  const theme = useContext(ThemeContext);
}
```

### Privacy & data boundaries in components
- Never hardcode real user data (names, emails, phone numbers) in component examples, fixtures, Storybook stories, or seed data — use obviously-fake placeholders (`jane.doe@example.com`, not a real person's address book entry). See [Documentation Standards](../../shared/documentation-standards.md) for the same rule applied to docs/examples generally.
- Don't wire analytics, tracking pixels, or session-replay snippets directly into a shared component — those are consent-gated concerns owned by [Analytics & Tracking](../analytics-tracking.md) and [Privacy & Data Compliance](../../shared/privacy-compliance.md), not something a component author adds ad hoc.

---

## Recommended (opt-in)

- **Barrel files (`index.ts`) per feature folder** to simplify imports — skip for large shared libraries where they hurt tree-shaking and build performance; measure before adopting broadly.

---

## Anti-Patterns (do not ship)

- Components organized by type (`components/hooks/`, `components/utils/` mixed in with UI) instead of by domain.
- `any`-typed or untyped props.
- A single component handling data fetching, business logic, and presentation with no separation.
- Fetching data inline with `useEffect` + `fetch` in a component when a dedicated data layer (a custom hook, React Query/SWR) would do — see [Hooks](hooks.md) and [State Management](state-management.md).
- Real user data (names, emails, addresses) in fixtures, stories, or examples.
- Analytics/tracking wired directly into a shared component instead of through the consent-gated tracking layer.

---

## Quick Reference

```
✓ Colocate feature-specific components; shared components organized by domain
✓ Typed props via interface/type, never any
✓ use() over useContext() (React 19); compound components for genuine multi-part UI
✓ Fake placeholder data only in examples/fixtures/stories
✗ No type-based component folders
✗ No inline useEffect data-fetching when a hook/library would do
✗ No real user data in examples; no ad hoc tracking inside shared components
```

---
*Section version: 0.1 — initial draft*
