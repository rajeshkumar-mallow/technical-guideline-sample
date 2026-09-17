# Testing (Next.js)

**Read this first.** This file covers Next.js-specific testing setup — mocking the App Router and testing Server Components/Actions. Component testing fundamentals (RTL, `renderHook`) live in [`../react/testing.md`](../react/testing.md) and apply here too; general testing principles live in [Testing Philosophy](../../shared/testing-philosophy.md).

**Applies to:** Component, integration, and end-to-end tests in a Next.js 15 App Router project.

---

## Mandatory

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

### Playwright against a production build
Run e2e against a production build (`next build && next start`), not `next dev`, so timing and bundling match reality. Include accessibility checks (`@axe-core/playwright`) on key flows.

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

---

## Recommended (opt-in)

- **Mock Service Worker (MSW)** for integration tests that need realistic network responses without hitting a real backend.
- **Visual regression testing** (Playwright snapshots or Chromatic) once the design system is stable enough that visual diffs are signal, not noise.

---

## Anti-Patterns (do not ship)

- Mocking `next/router` in an App Router project (wrong module — use `next/navigation`).
- Testing Server Components by trying to simulate their render with client-only tooling that doesn't support `async` components.
- Running Playwright e2e against `next dev` in CI as the only environment tested — dev mode has different performance/error characteristics than production.

## Quick Reference

```
✓ Mock next/navigation (not next/router) in App Router tests
✓ Server Actions tested as plain async functions
✓ Playwright e2e on a production build, primary user loop covered
✗ next/router mocked in App Router code
✗ e2e only against next dev
```

---
*Section version: 0.1 — initial draft*
