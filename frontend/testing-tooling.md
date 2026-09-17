# Testing Tooling (Frontend)

**Read this first.** This file names the specific tools for each layer of the test pyramid. For *why* we test at each layer and what coverage is expected, see [Testing Philosophy](../shared/testing-philosophy.md) first — this file is the "which tool, how configured" companion to that principle.

**Applies to:** All frontend projects (any framework). React/Next-specific test setup (mocking the router, RSC considerations) lives in [react-next/testing.md](../react-next/testing.md).

---

## Mandatory

### Unit tests — Vitest
- **Vitest** is the default unit-test runner for Vite-based projects: shares Vite's config/transform pipeline, so no separate Babel/webpack setup to maintain, and is significantly faster than Jest on the same suite.
- Jest remains acceptable (not preferred) where a project's toolchain is Jest-native for other reasons (e.g. an existing large suite, or a meta-framework with built-in Jest wiring) — record the choice, don't silently drift between the two across projects.
- Unit tests cover pure functions, utilities, hooks-in-isolation, and reducers/state logic — fast, no DOM required unless the unit under test needs one.

```typescript
// utils/currency.test.ts
import { describe, it, expect } from 'vitest';
import { formatCents } from './currency';

describe('formatCents', () => {
  it('formats whole dollars without decimals dropped', () => {
    expect(formatCents(1050)).toBe('$10.50');
  });
});
```

### Component / functional tests — Testing Library
- **Testing Library** (`@testing-library/react` or the framework-equivalent) for component tests, run under Vitest/Jest with `jsdom`/`happy-dom`.
- Test **behavior from the user's perspective** — query by role/label/text (`getByRole('button', { name: /submit/i })`), not by implementation detail (CSS class, internal state, `data-testid` as a last resort only).
- Every interactive component test covers: default render, the primary interaction, and at least one error/edge state (disabled, empty, loading).

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

test('shows a validation error when submitting an empty email', async () => {
  render(<LoginForm onSubmit={vi.fn()} />);
  await userEvent.click(screen.getByRole('button', { name: /sign in/i }));
  expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
});
```

### End-to-end tests — Playwright (default)
- **Playwright** is the default e2e tool: cross-browser (Chromium/Firefox/WebKit) in one API, built-in auto-waiting (fewer flaky sleeps), parallel execution, and first-class trace/video debugging on failure.
- **Cypress** is an acceptable alternative where a project already has an established Cypress suite — don't force a mid-project migration without a clear payoff; new projects default to Playwright.
- E2e suites cover the **primary user loop(s)** end-to-end against a real (or realistically stubbed) backend — not every edge case; that's what unit/component tests are for.
- Run e2e against a production-like build, not the dev server, before merge to main/release branches.

```typescript
// e2e/checkout.spec.ts
import { test, expect } from '@playwright/test';

test('user can complete checkout', async ({ page }) => {
  await page.goto('/cart');
  await page.getByRole('button', { name: /checkout/i }).click();
  await page.getByLabel(/card number/i).fill('4242424242424242');
  await page.getByRole('button', { name: /pay/i }).click();
  await expect(page.getByText(/order confirmed/i)).toBeVisible();
});
```

### Accessibility tests
- `axe-core` runs as part of component or e2e tests on key flows (see [Accessibility](accessibility.md)) — wired through `@axe-core/playwright` or `jest-axe`, not a separate manual-only process.

---

## Recommended (opt-in)

- Visual regression testing (Playwright's built-in screenshot comparison, or Chromatic against Storybook) once a project has a stable enough design system that visual diffs are signal, not noise.
- Mock Service Worker (`msw`) for intercepting network calls in component tests and Storybook — keeps tests decoupled from a live backend.
- Test coverage reporting via `@vitest/coverage-v8`, gated to the threshold set in [Testing Philosophy](../shared/testing-philosophy.md).

---

## Anti-Patterns (do not ship)

- Querying the DOM by CSS class or implementation detail instead of role/label/text.
- E2e suites that re-test every unit-level edge case (slow, brittle, wrong layer).
- `data-testid` used as the default query strategy instead of a last resort.
- Flaky tests silenced with `test.skip`/arbitrary `sleep()` instead of fixing the root cause (see the debt-tracking rule in the eventual [Versioning & Release Process](../shared/versioning-release-process.md) tie-in) — quarantine with a ticket, never silently retry-away.
- Running e2e only against the dev server, never a production-like build.

## Quick Reference

```
✓ Vitest default for unit (Jest acceptable if already established)
✓ Testing Library for component tests · query by role/label, not class/testid
✓ Playwright default for e2e (Cypress acceptable for existing suites)
✓ axe-core wired into component/e2e tests on key flows
✓ e2e covers the primary user loop against a prod-like build
✗ No querying by CSS class/implementation detail
✗ No data-testid as default query strategy
✗ No silently skipped/flaky tests
```

---
*Section version: 0.1 — initial draft*
