# State Management

**Read this first.** Reaching for the wrong state tool — global state for something local, client state for something the server already owns — is the most common source of React app complexity. This file gives a decision tree, not a single mandated library. It's framework-agnostic; Next.js-specific server-fetching mechanics (Server Component `fetch`, App Router caching) are in [`../next/routing-data-fetching.md`](../next/routing-data-fetching.md).

**Applies to:** any React codebase.

---

## Mandatory

### Decision tree

Ask in this order:

1. **Does the data live on the server (DB, API)?** → It's **server state**. Do not copy it into `useState`/Context. Use a server-state library for fetching/caching/mutation (**React Query** or **SWR**) — or, in a meta-framework with server-side data access (e.g. Next.js Server Components), fetch there directly.
2. **Is the state used by exactly one component (and maybe its direct children via props)?** → **Local state** (`useState`/`useReducer`). Default choice — don't globalize state prematurely.
3. **Is the state read/written by components in genuinely different parts of the tree, with no sensible prop-drilling path?** → **Global client state.** Reach for Context only for low-frequency-update values (theme, locale, auth session); reach for a state library (Zustand, Jotai) when updates are frequent or the state graph is nontrivial — Context re-renders every consumer on every change, which becomes a real perf problem for high-frequency state.

```
server data?  ──yes──▶ server state (React Query / SWR, or framework server-fetch)
     │ no
     ▼
single component/subtree? ──yes──▶ useState / useReducer
     │ no
     ▼
low-frequency, few consumers? ──yes──▶ React Context
     │ no
     ▼
frequent updates / complex graph ──▶ Zustand or Jotai (pick one, record via ADR)
```

### Local state
- `useState` for independent primitives/objects; `useReducer` once a component has **multiple pieces of state that change together** in response to the same actions (a reducer makes the valid transitions explicit instead of scattered `setX` calls).
- Never store server data (fetched lists, user records) in local state as a cache substitute — it goes stale silently with no revalidation.

### Global client state
- **Recommended default: Zustand** for cross-tree client state (auth UI state, cart, multi-step wizard state that outlives a single view) — minimal boilerplate, no provider-wrapping requirement, selective subscriptions avoid Context's re-render-everything problem. Record the chosen library via ADR if a project picks Jotai or another alternative instead — don't mix multiple global-state libraries in one codebase without a documented reason.
- **Context** stays appropriate for: theme, locale, auth session object (read-heavy, rarely-written), feature-flag values — values that change rarely and are read broadly.

```tsx
// store/cart-store.ts
import { create } from "zustand";

type CartState = {
  items: CartItem[];
  addItem: (item: CartItem) => void;
};

export const useCartStore = create<CartState>((set) => ({
  items: [],
  addItem: (item) => set((s) => ({ items: [...s.items, item] })),
}));
```

### Server state
- Data owned by the backend is fetched, cached, and revalidated by a library or framework mechanism — never manually synced into a global client store "for convenience." Duplicating server data into client state creates a second source of truth that drifts.
- Use **React Query** or **SWR** (pick one project-wide) for client-side fetch/refetch/mutation/optimistic updates rather than hand-rolled `useEffect` + `useState` fetching.

---

## Recommended (opt-in)

- **URL state** (via your router's search-params API) for anything that should be shareable/bookmarkable or survive a refresh — filters, pagination, active tab. Prefer it over component state for these cases.
- **`useOptimistic`** (React 19) for optimistic UI on low-risk mutations (a like button, a toggle) — not for anything that can fail validation server-side. See [Forms & Validation](forms-validation.md) for the mutation boundary.

---

## Anti-Patterns (do not ship)

- Copying fetched server data into `useState` and manually managing "loading"/"error" booleans by hand when a server-state library already solves this.
- Reaching for Context for high-frequency state (e.g. form field values, drag position) — causes broad re-renders.
- Running two different global-state libraries in the same codebase without an ADR justifying it.
- Prop-drilling more than 2–3 levels instead of composing components or reaching for Context/store.
- Global state for something only one component tree actually needs.

---

## Quick Reference

```
✓ Server data → React Query/SWR (or framework server-fetch), never copied into useState
✓ Local-first default: useState/useReducer
✓ Context for low-frequency broad reads; Zustand/Jotai for frequent/complex client state
✓ URL state for shareable/bookmarkable UI state
✗ No manual server-data mirroring into client state
✗ No Context for high-frequency updates
✗ No mixing multiple global-state libraries without an ADR
```

---
*Section version: 0.1 — initial draft*
