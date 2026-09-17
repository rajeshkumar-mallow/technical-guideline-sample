# Testing Philosophy

**Read this first.** This defines *what* to test, *how much*, and *why* — the shared principles behind every test suite. Tool-specific setup (which runner, which e2e framework, how to configure it) lives in [frontend/testing-tooling.md](../frontend/testing-tooling.md) and [react-next/testing.md](../react-next/testing.md) — this file does not duplicate that.

**Applies to:** every repo adopting this guideline, regardless of framework.

---

## Mandatory

### The test pyramid
```
        ▲
       /e2e\        Few. Slow. Cover the primary user loop only.
      /------\
     /  integ. \     Some. Component + a few real dependencies.
    /----------\
   /    unit     \   Many. Fast. Pure logic, isolated components.
  /--------------\
```
- **Unit**: pure functions, isolated component rendering, custom hooks in isolation. No network, no real timers, no DOM beyond what the component under test needs.
- **Functional / integration**: a component or page composed with its real children and a mocked API boundary — verifies the pieces work together, not implementation detail.
- **End-to-end**: a real browser driving the primary user loop(s) against a running app. Slowest, most valuable for catching what unit/integration miss (real navigation, real CSS, real network).

### What "done" means for a change
A PR that changes behavior is not done until:
1. The behavior change has a test that would fail without the fix/feature.
2. Existing tests still pass — a broken test is fixed at the root cause, not skipped.
3. Edge cases relevant to the change are covered: empty state, error state, loading state, boundary values.

### Test what the user observes, not implementation
- Query by role/label/text (what a user or screen reader sees), not by internal state, class names, or implementation details. This is enforced at the tool level in [react-next/testing.md](../react-next/testing.md) (Testing Library's guiding principle).
- A refactor that doesn't change behavior should not break tests. If it does, the test was coupled to implementation, not behavior.

### Coverage
- **≥ 80% line coverage**, enforced in CI, as a floor — not a target to chase for its own sake. 100% coverage with weak assertions is worse than 80% with meaningful ones.
- Coverage drops on a PR are treated as debt to justify or fix in the same PR, not deferred silently.
- Critical paths (auth, checkout/payment, data mutation) are held to a higher bar than coverage percentage alone: explicit test cases for both the success and failure path, not just "covered by an integration test that happens to touch it."

### What must always be tested
| Area | Requirement |
|---|---|
| Business logic / utilities | Every public function, including edge cases and error paths |
| Components with logic (not pure presentation) | Render states (loading/error/empty/success), user interaction outcomes |
| Custom hooks | Behavior across their state transitions |
| Forms | Validation rules (valid + each invalid case), submit success and failure |
| API/data-layer code | Success, error, and timeout/retry behavior (ties to [API Resilience](../frontend/api-resilience.md)) |
| Auth-gated UI/routes | Both authorized and unauthorized access |

### Flaky tests
- A flaky test is quarantined (skip + linked ticket + deadline) immediately, not silently retried into passing or left to intermittently fail CI for everyone. Never fix flakiness by adding an arbitrary `sleep`.

## Recommended (opt-in)

- **Snapshot tests** only for stable, low-churn output (e.g. a design-token map) — not for component markup, which produces noisy, low-signal diffs on every change.
- **Visual regression testing** (Chromatic, Percy) for a component library or design-system package — see [Design to Code](../frontend/design-to-code.md).
- **Mutation testing** (Stryker) periodically on core business logic to check whether the test suite would actually catch a real bug, not just execute the lines.
- **Contract tests** between frontend and backend API boundaries when both are owned in-house and drift is a recurring problem.

## Anti-Patterns (do not ship)

- Tests that assert on implementation details (internal state, private methods, CSS class names) instead of observable behavior.
- `it.skip` / `test.todo` left indefinitely with no ticket.
- Testing the framework/library itself (e.g. asserting `useState` updates state) instead of your code's behavior.
- One giant e2e suite as the only test coverage — slow feedback, and a single flaky step blocks everyone.
- Coverage-chasing tests with no real assertions (`expect(true).toBe(true)`) added purely to hit a number.

## Quick Reference

```
✓ Pyramid: many unit, some integration, few e2e (primary loop only)
✓ Test behavior a user observes, not implementation
✓ ≥80% coverage floor, enforced in CI — not a target to game
✓ Every change: test that fails without it; loading/error/empty states covered
✓ Flaky tests quarantined immediately with ticket + deadline
✗ No implementation-detail assertions · no indefinite .skip · no sleep-based flake fixes
```

---
*Section version: 0.1 — initial draft*
