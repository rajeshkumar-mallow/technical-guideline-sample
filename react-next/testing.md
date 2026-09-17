# Testing (React/Next)

**Read this first.** This file covers React/Next-specific testing setup — tool configuration and framework-specific mocking. General testing principles (pyramid, coverage targets, what "done" means) live in [shared/testing-philosophy.md](../shared/testing-philosophy.md); tool selection for plain frontend work lives in [frontend/testing-tooling.md](../frontend/testing-tooling.md).

**Applies to:** Component, integration, and end-to-end tests in a React/Next 15 App Router project.

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

### Mock `next/navigation`, not `next/router`
The App Router uses `next/navigation` (`useRouter`, `usePathname`, `useSearchParams`) — a different module from the Pages Router's `next/router`. Mock the one your code actually imports.

```ts
// __mocks__/next/navigation.ts or a test setup file
vi.mock("next/navigation", () => ({
  useRouter: () => ({ push: vi.fn(), replace: vi.fn(), refresh: vi.fn() }),
  usePathname: () => "/dashboard",
  useSearchParams: () => new URLSearchParams(),
}));
```

Prefer asserting on the mock's `push`/`replace` calls over asserting on rendered URL state, unless the URL itself is the thing under test.

### Server Components: test what they produce, not how
A Server Component is an async function returning JSX — test it by `await`-ing the component and rendering the result, or, more commonly, test it indirectly through an integration/e2e test since it depends on server-only data access. Don't try to shallow-render it like a Client Component.

```tsx
// Direct unit test of an async Server Component
test("renders the product name", async () => {
  const ui = await ProductPage({ params: { id: "123" } });
  render(ui);
  expect(screen.getByRole("heading")).toHaveTextContent("Widget");
});
```

For anything that mixes Server + Client Components in a real data flow (most pages), rely on **Playwright** against a running app rather than fighting the boundary in a unit test.

### Playwright for end-to-end
The primary user loop (signup, core workflow, checkout — whatever the product's critical path is) is covered end-to-end against a real running Next build, not a mocked environment.

```ts
import { test, expect } from "@playwright/test";

test("user can complete signup", async ({ page }) => {
  await page.goto("/signup");
  await page.getByLabel("Email").fill("test@example.com");
  await page.getByLabel("Password").fill("a-strong-password-123");
  await page.getByRole("button", { name: "Sign up" }).click();
  await expect(page).toHaveURL("/onboarding");
});
```

- Run against a production build (`next build && next start`), not `next dev`, so timing and bundling match reality.
- Include accessibility checks (`@axe-core/playwright`) on key flows — ties to [frontend/accessibility.md](../frontend/accessibility.md).

### Server Actions: test as functions
A Server Action is a plain async function — call it directly with a `FormData` or typed argument in a unit test; no need to render a form to exercise its validation/error paths.

```ts
test("rejects an invalid signup payload", async () => {
  const fd = new FormData();
  fd.set("email", "bad");
  const result = await signupAction(undefined, fd);
  expect(result.error?.email).toBeDefined();
});
```

---

## Recommended (opt-in)

- **Mock Service Worker (MSW)** for integration tests that need realistic network responses without hitting a real backend.
- **Visual regression testing** (Playwright snapshots or Chromatic) once the design system is stable enough that visual diffs are signal, not noise.
- **Component testing in Playwright** (`@playwright/experimental-ct-react`) as an alternative to RTL when a component genuinely needs a real browser environment (e.g. canvas, real layout measurement).

---

## Anti-Patterns (do not ship)

- Mocking `next/router` in an App Router project (wrong module — use `next/navigation`).
- Snapshot-testing entire pages as the primary assertion strategy — brittle, low signal; prefer targeted behavioral assertions.
- Testing Server Components by trying to simulate their render with client-only tooling that doesn't support `async` components.
- Running Playwright e2e against `next dev` in CI as the only environment tested — dev mode has different performance/error characteristics than production.
- Asserting on CSS class names or DOM structure instead of accessible roles/labels.

## Quick Reference

```
✓ RTL: query by role/label, user-event for interaction
✓ Mock next/navigation (not next/router) in App Router tests
✓ Server Actions tested as plain async functions
✓ Playwright e2e on a production build, primary user loop covered
✓ axe-core checks on key flows
✗ next/router mocked in App Router code
✗ Page-level snapshot tests as primary coverage
✗ e2e only against next dev
```

---
*Section version: 0.1 — initial draft*
