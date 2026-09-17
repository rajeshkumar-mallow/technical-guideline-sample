# Hooks

**Read this first.** Custom hooks are how shared stateful logic stays testable and composable instead of duplicated across components. This file covers naming, enforcement, and when extraction is actually worth it. Fully framework-agnostic.

**Applies to:** any React codebase.

---

## Mandatory

### Rules of Hooks enforcement
- **`eslint-plugin-react-hooks` is mandatory**, its rules run as errors (not warnings) in CI — violations of the Rules of Hooks (conditional hooks, hooks in loops, hooks called outside components/hooks) are correctness bugs, not style issues, and must block merge.
- Hooks are only called at the top level of a component or another hook — never inside conditionals, loops, or nested functions.

### Naming
- Every custom hook name starts with `use` (enforced by the lint rule, which also relies on this prefix to know what to check) — `useDebounce`, `useLocalStorage`, `useFormField`.
- The name describes **what it returns/manages**, not the implementation: `useCartTotal`, not `useReduceCartItems`.

### When to extract a custom hook
Extract when **two or more** of these are true:
- The same stateful logic (state + effect + handler) is duplicated in more than one component.
- The logic is non-trivial enough that inlining it obscures the component's actual render logic (a component should read as "what it renders," not "how it manages its subscription").
- The logic needs independent unit testing separate from any component that uses it.

Do **not** extract a hook for a single `useState` call or a one-line derived value — that's premature abstraction; a plain variable or inline `useMemo` is clearer.

```tsx
// Worth extracting: encapsulates subscribe/unsubscribe + state, reused in 3+ places
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  useEffect(() => {
    const on = () => setIsOnline(true);
    const off = () => setIsOnline(false);
    window.addEventListener("online", on);
    window.addEventListener("offline", off);
    return () => {
      window.removeEventListener("online", on);
      window.removeEventListener("offline", off);
    };
  }, []);
  return isOnline;
}

// NOT worth extracting — just inline it
const [isOpen, setIsOpen] = useState(false);
```

### Hook contracts
- A custom hook's return shape is **explicit and typed** — return an object with named keys for 3+ values (not a long positional tuple), a tuple only for 2 tightly-coupled values (mirroring `useState`'s own convention).
- Side effects inside a hook clean up after themselves (`useEffect` return function) — no dangling subscriptions, timers, or listeners.
- Hooks that fetch or subscribe to server data use the project's server-state library (React Query/SWR) internally rather than hand-rolled `useEffect` fetching — see [`state-management.md`](state-management.md).

---

## Recommended (opt-in)

- Co-locate a hook with its sole consumer (a `hooks/` folder scoped to that feature) when it's feature-specific; promote to a shared top-level `hooks/` folder only once a second consumer appears.
- Unit test custom hooks with `@testing-library/react`'s `renderHook` for any hook with nontrivial branching logic — see [`testing.md`](testing.md).

---

## Anti-Patterns (do not ship)

- Hooks called conditionally (`if (x) { useEffect(...) }`) or inside loops.
- A "god hook" that manages five unrelated concerns because it grew feature-by-feature — split by responsibility.
- Extracting a hook for a single `useState` with no other logic attached.
- Missing cleanup functions in effects that subscribe/add listeners/set timers.
- Hand-rolled fetch-in-`useEffect` logic duplicated across components instead of using the project's server-state library.

---

## Quick Reference

```
✓ eslint-plugin-react-hooks errors in CI, not warnings
✓ use-prefixed names describing what the hook returns
✓ Extract when logic is duplicated, non-trivial, or needs isolated testing
✓ Explicit typed return shape; effects clean up after themselves
✗ No conditional/looped hook calls
✗ No god hooks bundling unrelated concerns
✗ No hook extraction for a single trivial useState
```

---
*Section version: 0.1 — initial draft*
