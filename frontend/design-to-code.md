# Design to Code

**Read this first.** The gap between a Figma file and shipped code is where visual drift, missed states, and rebuilt-from-scratch components come from. Treat design tokens as the contract between design and engineering, not the pixel-perfect screenshot.

**Applies to:** Any project with a dedicated design phase (Figma or equivalent) preceding implementation.

---

## Mandatory

### Design tokens are the source of truth, not eyeballed values
Colors, spacing, type scale, radii, and shadows are defined once in Figma (as Figma Variables/Styles) and exported to the codebase as the same [design tokens](styling-foundations.md#design-tokens-as-css-custom-properties) engineering consumes — not re-guessed from a screenshot with a color picker.

- Figma variable names map 1:1 (or via a documented transform) to CSS custom property / Tailwind theme names — `color/primary/600` → `--color-primary-600`.
- A token change in Figma is a diffable, reviewable change in the codebase (via manual sync or an export pipeline), not silent drift.

### Component handoff checklist
Before implementation starts on a component, the design file provides:

- [ ] All interactive **states** — default, hover, focus-visible, active, disabled, loading, error (missing states are the #1 cause of "figure it out yourself" during implementation)
- [ ] **Responsive behavior** — at minimum mobile + desktop frames, or explicit notes on what adapts
- [ ] **Spacing/sizing** as tokens or explicit values, not "eyeball it"
- [ ] **Content edge cases** — empty state, long text overflow/truncation behavior, max item counts
- [ ] **Accessibility notes** where non-obvious — reading order for complex layouts, what's announced to screen readers for dynamic updates

A component missing states or edge cases goes back to design before implementation starts — don't let engineering silently invent the missing states.

### Storybook (or equivalent) is required for shared components
Any component that lives in a shared/design-system package — not a one-off page section — gets a Storybook story (or equivalent isolated-render tool) covering its documented states. This is the review surface for design sign-off and the regression surface for visual testing below.

```ts
// Button.stories.tsx
export const Default: Story = { args: { variant: 'primary' } }
export const Disabled: Story = { args: { variant: 'primary', disabled: true } }
export const Loading: Story = { args: { variant: 'primary', loading: true } }
```

## Recommended (opt-in)

- **Visual regression testing** (Chromatic, Percy, or Playwright screenshot comparison) on the Storybook/component catalog once the shared component set is large enough that manual visual review misses regressions — not needed for a two-page brochure site.
- **Figma dev-mode / Code Connect** style tooling to let engineers inspect exact spacing/token values directly from the design file, reducing handoff meetings.
- **A living style guide page** in the app itself (not just Storybook) when non-engineers (PM, QA) need to reference current component states without dev tooling access.

## Anti-Patterns (do not ship)

- Implementing colors/spacing by eyeballing a Figma screenshot instead of reading the token value.
- Shipping a shared component with only the "happy path" state implemented, backfilling hover/error/loading states after they're noticed missing in production.
- A design system component with no Storybook story — no isolated way to review or test it.
- Token values drifting between Figma and code with no process to catch it (design updates a color, code silently keeps the old one).

## Quick Reference
```
✓ Tokens exported from Figma variables, not eyeballed
✓ Handoff checklist complete before implementation starts (states, responsive, edge cases, a11y notes)
✓ Shared components have a Storybook story per documented state
✗ No eyeballed pixel/color values from screenshots
✗ No shared component shipped without documented states
✗ No silent design-token drift between Figma and code
```

---
*Section version: 0.1 — initial draft*
