# Responsive Design

**Read this first.** Mobile-first isn't just "test it on a phone at the end" — it's a CSS authoring order that produces simpler, more maintainable stylesheets than desktop-first override chains. Accessibility target (§[accessibility.md](accessibility.md)) stays mandatory regardless of whether mobile is a primary target.

**Applies to:** All layout and interactive-target sizing across the app.

---

## Mandatory

### Mobile-first CSS
Base styles target the smallest viewport; `min-width` media queries add complexity as the viewport grows. Never write desktop styles first and override down with `max-width` queries — it produces more overrides, not fewer.

```css
/* Mobile-first: base styles are the small-viewport case */
.card-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--space-4);
}

@media (min-width: 768px) {
  .card-grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1024px) {
  .card-grid { grid-template-columns: repeat(3, 1fr); }
}
```

### Standard breakpoint scale
Use one shared breakpoint scale project-wide (Tailwind defaults are a reasonable baseline) — don't invent ad-hoc breakpoints per component.

| Token | Width | Typical target |
|---|---|---|
| `sm` | 640px | Large phone / small tablet portrait |
| `md` | 768px | Tablet |
| `lg` | 1024px | Small laptop |
| `xl` | 1280px | Desktop |
| `2xl` | 1536px | Large desktop |

### Touch target sizing
Interactive elements (buttons, links, form controls, icon-only actions) are **≥ 44×44px** touch targets, even on desktop-first products — a smaller visual element gets invisible padding to reach the minimum. This is a WCAG 2.1 AA-adjacent requirement (2.5.5 Target Size), not just a mobile nicety.

```css
.icon-button {
  min-width: 44px;
  min-height: 44px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
```

### Interaction states beyond hover
Hover-only affordances don't exist on touch devices. Every interactive element has a visible **`:active`** state for touch feedback and a visible **`:focus-visible`** state for keyboard users — hover (`:hover`) is progressive enhancement on top, never the only signal.

```css
.button {
  background: var(--color-primary);
}
.button:hover { background: var(--color-primary-dark); }     /* pointer only, enhancement */
.button:active { background: var(--color-primary-darker); }  /* touch feedback */
.button:focus-visible { outline: 2px solid var(--color-primary); outline-offset: 2px; } /* keyboard */
```

### Viewport units that don't break on mobile
Use `dvh`/`svh` (dynamic/small viewport height) instead of `vh` for full-height layouts — `100vh` overflows behind mobile browser chrome (address bar) on iOS/Android.

```css
.full-height-section { min-height: 100dvh; } /* not 100vh */
```

### Responsive pattern: tables → cards
Data tables collapse to a card/stacked layout below `md` rather than forcing horizontal scroll on a data grid not designed for it — horizontal-scroll tables are acceptable only when the data is genuinely tabular and scroll affordance is obvious (scroll shadow/indicator).

## Recommended (opt-in)

- **Container queries** (`@container`) once a component's layout needs to respond to its container's size rather than the viewport (e.g. a card that's sometimes in a sidebar, sometimes full-width) — more precise than viewport breakpoints for reusable components.
- **`clamp()`-based fluid sizing** for type/spacing that scales smoothly between breakpoints instead of stepping at each one — see [`styling-foundations.md`](styling-foundations.md).
- **Mobile system tests at a real viewport** (e.g. 390×844) for the primary user loop, once mobile is a first-class target — not required for an internal admin tool used only on desktop.

## Anti-Patterns (do not ship)

- Desktop-first CSS with `max-width` override chains growing more complex than mobile-first `min-width` would have been.
- Touch targets smaller than 44×44px on any interactive element, including icon-only buttons.
- Hover-only affordances with no `:active`/`:focus-visible` equivalent — a control that's invisible or inoperable without a mouse.
- `100vh` used for full-height mobile layouts, causing content to be hidden behind browser chrome.
- Wide data tables forced into horizontal scroll on mobile with no card/stacked alternative and no scroll affordance.

## Quick Reference
```
✓ Mobile-first CSS: base = small viewport, min-width queries add complexity up
✓ One shared breakpoint scale project-wide
✓ Touch targets ≥ 44×44px
✓ active: and focus-visible: states on every interactive element, not hover-only
✓ dvh/svh over vh for full-height layouts
✓ Tables → cards below md
✗ No desktop-first max-width override chains
✗ No touch targets under 44×44px
✗ No hover-only affordances
```

---
*Section version: 0.1 — initial draft*
