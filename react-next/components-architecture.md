# Components & Architecture

**Read this first.** The Server/Client Component boundary is the single most consequential architectural decision in a Next.js App Router codebase — get it wrong and you either ship a bloated client bundle or lose the interactivity you need. This file sets folder conventions and boundary rules; general component-design principles live in [`frontend/html-css.md`](../frontend/html-css.md) and [`frontend/javascript.md`](../frontend/javascript.md).

**Applies to:** Next.js App Router projects.

---

## Mandatory

### Server Components by default
- Every component is a **Server Component unless it needs client interactivity** (state, effects, browser APIs, event handlers) or a client-only library. Do not add `"use client"` preemptively "just in case."
- `"use client"` is placed at the **leaf** of the tree, as close as possible to the interactive element — not at a layout or page root, which would force everything beneath it into the client bundle.

```tsx
// app/dashboard/page.tsx — Server Component, fetches data directly
export default async function DashboardPage() {
  const data = await getDashboardData();
  return (
    <DashboardLayout>
      <StatsSummary data={data} />         {/* Server Component */}
      <RefreshButton />                     {/* Client Component — leaf */}
    </DashboardLayout>
  );
}

// components/refresh-button.tsx
"use client";
export function RefreshButton() {
  const [pending, startTransition] = useTransition();
  return <button onClick={() => startTransition(refresh)}>Refresh</button>;
}
```

- **Composition over lifting the boundary up**: pass Server Components as `children`/props into Client Components rather than converting the whole subtree to client, when only a wrapper needs interactivity.

```tsx
// Client Component wraps Server Component children — children stay server-rendered
"use client";
function ExpandablePanel({ children }: { children: React.ReactNode }) {
  const [open, setOpen] = useState(false);
  return <div onClick={() => setOpen(!open)}>{open && children}</div>;
}
```

### Folder conventions
- **Colocate** route-specific components inside the route segment (`app/dashboard/_components/`) — the leading underscore excludes the folder from routing.
- **Shared/reusable** components live in a top-level `components/` (or `src/components/`), organized by domain, not by type (`components/billing/`, not `components/buttons/` + `components/forms/` split arbitrarily).
- One component per file; filename matches the component name (`invoice-table.tsx` exports `InvoiceTable`).

### Props & typing
- Every component's props are an explicit `interface` or `type`, never `any`, never untyped destructuring from an inferred object (see [`typescript-conventions.md`](../shared/typescript-conventions.md) for the broader TS baseline).
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

---

## Recommended (opt-in)

- **Barrel files (`index.ts`) per feature folder** to simplify imports — skip for large shared libraries where they hurt tree-shaking and build performance; measure before adopting broadly.
- **Compound components** (`<Tabs><Tabs.List><Tabs.Panel>`) for complex, multi-part UI — adopt when a feature has genuine internal composition needs, not as a default pattern.

---

## Anti-Patterns (do not ship)

- `"use client"` on a `layout.tsx` or `page.tsx` root when only one small piece of it is interactive.
- Fetching data in a Client Component with `useEffect` when the same data could be fetched server-side and passed down.
- Components organized by type (`components/hooks/`, `components/utils/` mixed in with UI) instead of by domain.
- `any`-typed or untyped props.
- A single component handling data fetching, business logic, and presentation with no separation.

---

## Quick Reference

```
✓ Server Component by default; "use client" only at interactive leaves
✓ Compose Server Components as children into Client wrappers
✓ Colocate route-specific components; shared components organized by domain
✓ Typed props via interface/type, never any
✗ No "use client" at layout/page root "just in case"
✗ No useEffect data-fetching when server fetch is available
✗ No type-based component folders
```

---
*Section version: 0.1 — initial draft*
