# Accessibility

**Read this first.** Accessibility is a mandatory baseline, not a nice-to-have add-on — it applies to every project regardless of target audience, device, or whether a customer has explicitly asked for it. Inaccessible UI is a shipped defect.

**Applies to:** All user-facing frontend surfaces (any framework).

---

## Mandatory

### Target: WCAG 2.1 AA
Every shipped flow meets **WCAG 2.1 Level AA**, regardless of target device. This is non-negotiable baseline, the same way the Rails guideline treats it as mandatory for server-rendered views. AAA is not required; AA violations block merge.

### Semantic HTML first
- Use the native element for the job (`button`, `a`, `nav`, `header`, `main`, `dialog`, `table`) before reaching for a generic `div`/`span` + ARIA. Native elements come with keyboard behavior, roles, and focus management for free.
- **ARIA only where native semantics fall short** — custom widgets (comboboxes, tab panels, menus) that have no native HTML equivalent. Follow the [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/) patterns exactly; don't improvise roles.
- Every image conveying meaning has descriptive `alt` text; purely decorative images use `alt=""` (not omitted `alt`).

### Labels & names
- Every form control has a **programmatic label** (`<label for>`, `aria-label`, or `aria-labelledby`) — placeholder text is never a substitute for a label.
- Every interactive control has an accessible name that describes its action, not its icon (`aria-label="Delete item"`, not `aria-label="Trash icon"`).

### Keyboard operability
- Every interactive element is reachable and operable via keyboard alone: `Tab`/`Shift+Tab` to move, `Enter`/`Space` to activate, `Esc` to dismiss overlays, arrow keys within composite widgets (menus, tabs, listboxes) per the APG pattern.
- **No keyboard traps** — a user must always be able to tab out of a widget (modals excepted, which trap focus deliberately and release it on close).
- Tab order follows visual/logical reading order. Avoid positive `tabindex` values; use `tabindex="0"`/`"-1"` only to include/exclude non-native-focusable elements.

### Visible focus states
- Every focusable element has a **visible focus indicator** distinguishable from its resting state — never `outline: none` without a replacement focus style.
- Use `:focus-visible` so focus rings appear for keyboard users without flashing on every mouse click.

### Color contrast
- Text and meaningful UI: **≥ 4.5:1** contrast ratio against its background (normal text); **≥ 3:1** for large text (≥ 24px, or ≥ 19px bold) and for graphical/UI component boundaries (icons, input borders).
- Never convey information (error state, required field, status) by color alone — pair it with text, an icon, or a pattern.

### Motion — `prefers-reduced-motion`
- Any non-essential animation (parallax, auto-playing carousels, large transitions) is disabled or reduced when the user has `prefers-reduced-motion: reduce` set:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

- Essential motion that conveys state (a loading spinner) may remain, but should still avoid strobing/flashing content (no more than 3 flashes per second — seizure risk).

### Automated testing — mandatory before merge
- Run **`axe-core`** (via `jest-axe`, `@axe-core/playwright`, or equivalent) against key flows in CI. **Zero AA violations before merge.**
- Automated testing catches roughly a third of issues — it supplements, not replaces, manual keyboard and screen-reader spot checks on new interactive components.

```javascript
// Example: axe-core in a Playwright test
import AxeBuilder from '@axe-core/playwright';

test('checkout page has no AA violations', async ({ page }) => {
  await page.goto('/checkout');
  const results = await new AxeBuilder({ page }).withTags(['wcag2a', 'wcag2aa']).analyze();
  expect(results.violations).toEqual([]);
});
```

See [Testing Tooling](testing-tooling.md) for where this fits in the overall test suite.

---

## Recommended (opt-in)

- Manual screen-reader pass (VoiceOver/NVDA) on new, non-trivial custom widgets before shipping.
- A documented component-level a11y checklist in Storybook (see [Design to Code](design-to-code.md)) so reviewers can verify state variants (focus, error, disabled) visually.
- Skip-to-content link for pages with heavy repeated navigation.

---

## Anti-Patterns (do not ship)

- `outline: none` with no replacement focus style.
- `<div onClick>` used as a button — no keyboard access, no semantics.
- Placeholder-as-label on form fields.
- Auto-playing motion/video with no way to pause and no `prefers-reduced-motion` handling.
- Conveying required/error/success state by color alone.
- Positive `tabindex` values reordering tab flow.
- Shipping a custom widget (dropdown, modal, tabs) without following its APG keyboard pattern.

## Quick Reference

```
✓ WCAG 2.1 AA mandatory, every flow, every device
✓ Semantic HTML first · ARIA only where native falls short
✓ Every control has a programmatic label + accessible name
✓ Full keyboard operability · no traps · visible focus-visible states
✓ Contrast ≥ 4.5:1 text / ≥ 3:1 large text & UI boundaries
✓ prefers-reduced-motion honored for non-essential animation
✓ axe-core on key flows in CI · zero AA violations before merge
✗ No outline:none without replacement · no div-as-button · no placeholder-as-label
✗ No color-only state signaling
```

---
*Section version: 0.1 — initial draft*
