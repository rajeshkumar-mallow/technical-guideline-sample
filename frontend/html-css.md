# HTML & CSS

**Read this first.** Semantic markup and a consistent CSS naming approach are the foundation everything else (accessibility, styling foundations, performance) builds on. Get this wrong and every later layer inherits the debt.

**Applies to:** All markup and stylesheets, regardless of templating engine or framework.

---

## Mandatory

### Semantic HTML first
Use the element that matches the content's meaning before reaching for a generic `<div>`/`<span>` + ARIA. Native semantics give you keyboard behavior, screen-reader roles, and browser affordances for free — ties directly into [`accessibility.md`](accessibility.md).

| Need | Use | Not |
|---|---|---|
| Clickable action | `<button>` | `<div onclick>` |
| Navigation link | `<a href>` | `<span onclick>` |
| Page landmarks | `<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>` | generic `<div class="header">` |
| Grouped form fields | `<fieldset>` + `<legend>` | bare `<div>` with a visual label only |
| Tabular data | `<table>` (with `<th scope>`) | CSS-grid `<div>` soup for real tabular data |
| Headings | `<h1>`–`<h6>` in a logical, non-skipping order | styled `<div>`/`<span>` sized to look like a heading |

Only reach for ARIA roles when no native element provides the semantics (`role="tablist"` for a custom tab widget) — "no ARIA is better than bad ARIA."

### Document structure
- One `<h1>` per page (or per major view in an SPA); heading levels don't skip (`h2` → `h4` with no `h3` is a violation).
- Landmarks (`<main>`, `<nav>`) appear exactly where they semantically apply — not one `<main>` wrapping the entire app shell including the nav.
- `<img>` always has `alt` (empty `alt=""` for purely decorative images, descriptive text otherwise — never omitted).

### CSS naming methodology — pick one, don't mix
Choose per project and record the choice in the project addendum; don't let both conventions coexist in the same codebase.

**Utility-first (Tailwind or equivalent) — recommended default** for most product UI: colocates styling with markup, avoids naming bikeshedding, scales well with component-based frameworks.
```html
<button class="inline-flex items-center gap-2 rounded-md bg-primary px-4 py-2 text-sm font-medium text-white hover:bg-primary-dark">
  Save changes
</button>
```

**BEM (Block\_\_Element--Modifier)** when the project has genuinely custom, hand-written CSS at scale (marketing sites, design-system-authoring repos) and a utility framework doesn't fit:
```css
.card { }
.card__title { }
.card__title--large { }
.card--highlighted { }
```

Either way: styling variables/tokens (colors, spacing, radii) come from [`styling-foundations.md`](styling-foundations.md), not hardcoded per component.

### File & selector organization
- One stylesheet/style-module scope per component — no editing a 3,000-line global CSS file to change one component's look.
- No ID selectors for styling (`#header { }`) — IDs are for anchors/ARIA references/JS hooks, not CSS specificity.
- No `!important` outside of a narrowly justified utility override (documented with a comment on why).
- Nesting depth capped at ~3 levels (Sass/CSS nesting) — deeper nesting signals the component should be split.

## Recommended (opt-in)

- **CSS Modules or scoped styles** (`Component.module.css`, styled-components, vanilla-extract) when not using a utility framework, to get automatic scoping without BEM's manual discipline.
- **Stylelint** with a shared config to enforce the chosen methodology automatically rather than relying on review — pairs with [`../shared/code-style-linting.md`](../shared/code-style-linting.md).

## Anti-Patterns (do not ship)

- `<div>`/`<span>` with `onclick` standing in for `<button>`/`<a>` — breaks keyboard access and screen readers.
- Mixing BEM and utility classes in the same component without a reason.
- Inline `style=""` attributes for anything beyond a genuinely dynamic, computed value (e.g. a chart's data-driven width).
- Heading levels chosen for visual size instead of document structure (`<h2>` used because it "looks right," skipping `<h1>`).
- `!important` used to win a specificity fight instead of fixing the specificity.

## Quick Reference
```
✓ Semantic element before div+ARIA
✓ One h1, non-skipping heading order, alt on every img
✓ One CSS naming methodology per project (utility-first default, BEM when justified)
✓ Design tokens for color/spacing/radii — not hardcoded values
✓ Component-scoped styles, no global stylesheet edits for local changes
✗ No div/span standing in for button/a
✗ No ID selectors for styling, no unjustified !important
✗ No mixed BEM + utility in the same codebase
```

---
*Section version: 0.1 — initial draft*
