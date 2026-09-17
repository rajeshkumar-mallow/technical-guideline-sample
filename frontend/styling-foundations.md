# Styling Foundations

**Read this first.** These are the load-bearing decisions every component styling choice sits on top of: what resets the browser's defaults, what processes your CSS, where your design values live, and whether you build UI primitives or buy them. Get these right once, at project start — they're expensive to change later.

**Applies to:** Project-wide styling setup, decided once per project and recorded in the project addendum.

---

## Mandatory

### CSS reset / normalize
Every project includes a reset. Two acceptable options:

- **Modern minimal reset** (recommended default) — a small, current reset (e.g. Josh Comeau's or Andy Bell's modern CSS reset) that fixes box-sizing, margin, and media-element defaults without wiping useful native styling (like form control appearance where you still want it).
- **Framework-provided reset** — if using Tailwind, its Preflight layer covers this; don't add a second reset on top.

Never ship with zero reset — inconsistent default margins/box-sizing across browsers is a recurring, avoidable bug source.

```css
*, *::before, *::after { box-sizing: border-box; }
body { margin: 0; }
img, picture, video, canvas, svg { display: block; max-width: 100%; }
```

### Preprocessor / styling approach — decision table

| Approach | Choose when | Avoid when |
|---|---|---|
| **Tailwind CSS (recommended default)** | Component-based frontend (React/Next/Vue), team wants speed and consistency without a design-token discipline problem, no strong existing CSS architecture to preserve | Team has zero appetite for utility classes in markup, or project is a small marketing site where hand-written CSS is simpler |
| **PostCSS + plain CSS (custom properties, `@layer`, nesting)** | Team wants standards-based CSS without a build-time DSL; modern browser support is sufficient (autoprefixer covers gaps) | Need Sass-specific features (loops, maps for complex token generation) not yet standard in CSS |
| **Sass/SCSS** | Legacy codebase already on it; team needs advanced logic (mixins, loops) not natively in CSS yet | Greenfield project — plain CSS + PostCSS now covers most of what Sass used to be needed for |

Record the choice in the project addendum. Don't run two of these in the same codebase.

### Design tokens as CSS custom properties
Design values (color, spacing, radius, shadow, font scale) are defined once as CSS custom properties (or Tailwind theme config, which compiles to the same idea) and referenced everywhere — never hardcoded per component.

```css
:root {
  --color-primary: #2563eb;
  --color-primary-dark: #1d4ed8;
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --radius-md: 0.375rem;
  --font-size-base: 1rem;
}
```

Tokens are the single source of truth shared with design — see [`design-to-code.md`](design-to-code.md) for how they sync from Figma. Dark mode / theming is implemented by swapping token values (`[data-theme="dark"]`), not by duplicating component styles.

### UI component library — decision criteria
Don't default to "build everything bespoke" or "always pull in a full component library" — decide per project against these triggers:

| Signal | Lean toward |
|---|---|
| Small surface area, highly custom brand/visual design | **Headless library** (Radix UI, React Aria, Headless UI) + your own styling — gets you correct accessibility/keyboard behavior without fighting a pre-styled look |
| Internal tool / admin panel, speed matters more than bespoke visuals | **Full component library** (MUI, Chakra, Ant Design) — accept its visual defaults |
| Team already has (or is building) a design system with a distinct visual identity | **Headless + design tokens**, or a shared internal component package once patterns stabilize across projects |
| Genuinely tiny surface (a handful of components) | **Bespoke**, no library — the dependency isn't worth it yet |

Any choice outside "headless library for custom brand, full library for internal tools" needs a one-line rationale in the project addendum, not a full ADR.

## Recommended (opt-in)

- **`clamp()`-based fluid type/spacing scales** instead of many fixed breakpoint overrides, once the design calls for smooth scaling rather than stepped values.
- **A token-generation pipeline** (Style Dictionary or similar) once tokens need to feed multiple platforms (web + a native app) from one source, not just CSS.

## Anti-Patterns (do not ship)

- Hardcoded hex colors / pixel spacing values scattered through component files instead of tokens.
- Running Tailwind and a separate hand-rolled BEM system simultaneously with no clear boundary.
- Pulling in a full styled component library (MUI, Ant) for a project with a highly custom brand, then fighting its defaults with overrides everywhere.
- No reset at all, relying on "it looks fine in Chrome."
- Dark mode implemented as a second copy of every component's styles instead of token swapping.

## Quick Reference
```
✓ A reset/normalize present (modern minimal reset, or framework Preflight)
✓ One preprocessor/styling approach per project, recorded in project addendum
✓ Design tokens as CSS custom properties — colors/spacing/radii never hardcoded
✓ UI library choice matches the decision table (headless vs. full vs. bespoke)
✗ No hardcoded design values bypassing tokens
✗ No mixed styling approaches in one codebase
✗ No full-styled library fighting a custom brand without a documented reason
```

---
*Section version: 0.1 — initial draft*
