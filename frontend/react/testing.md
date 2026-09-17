# Testing (React)

**Read this first.** This file covers React component testing — tool usage, not tool selection. General testing principles (pyramid, coverage targets, what "done" means) live in [Testing Philosophy](../../shared/testing-philosophy.md); tool selection lives in [Testing Tooling](../../js/testing-tooling.md). Next.js-specific mocking (`next/navigation`, Server Components, Playwright against a Next build) is in [`../next/testing.md`](../next/testing.md).

**Applies to:** any React codebase.

---

## Mandatory

### React Testing Library for components
Test components through their rendered output and user interaction, not implementation details. Query by role/label/text — not by CSS class or internal state.

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { SignupForm } from "./SignupForm";

test("shows a validation error for an invalid email", async () => {
  render(<SignupForm />);
  await userEvent.type(screen.getByLabelText(/email/i), "not-an-email");
  await userEvent.click(screen.getByRole("button", { name: /sign up/i }));
  expect(await screen.findByRole("alert")).toHaveTextContent(/valid email/i);
});
```

### Testing custom hooks
Use `renderHook` from `@testing-library/react` for any hook with nontrivial branching logic — don't force a hook through a throwaway wrapper component just to call it. See [Hooks](hooks.md).

### End-to-end for the primary user loop
The primary user loop (signup, core workflow, checkout — whatever the product's critical path is) is covered end-to-end against a real running build with **Playwright**, not a mocked environment. Include accessibility checks (`@axe-core/playwright`) on key flows — ties to [Accessibility](../accessibility.md).

---

## Recommended (opt-in)

- **Mock Service Worker (MSW)** for integration tests that need realistic network responses without hitting a real backend.
- **Visual regression testing** (Playwright snapshots or Chromatic) once the design system is stable enough that visual diffs are signal, not noise.
- **Component testing in Playwright** (`@playwright/experimental-ct-react`) as an alternative to RTL when a component genuinely needs a real browser environment (e.g. canvas, real layout measurement).

---

## Anti-Patterns (do not ship)

- Snapshot-testing entire pages/components as the primary assertion strategy — brittle, low signal; prefer targeted behavioral assertions.
- Asserting on CSS class names or DOM structure instead of accessible roles/labels.
- Testing a custom hook only indirectly through a component when `renderHook` would isolate it directly.
- Running e2e only against a dev server in CI as the only environment tested — dev mode has different performance/error characteristics than production.

## Quick Reference

```
✓ RTL: query by role/label, user-event for interaction
✓ renderHook for testing custom hooks in isolation
✓ Playwright e2e on a production build, primary user loop covered
✓ axe-core checks on key flows
✗ Page/component-level snapshot tests as primary coverage
✗ CSS-class/DOM-structure assertions instead of roles/labels
✗ e2e only against a dev server
```

---
*Section version: 0.1 — initial draft*
