# Browser & Device Support Matrix

**Read this first.** "Support all browsers" is not a spec — it's a way to ship untested code. This file defines the default supported matrix, how degradation is handled outside it, and how a project records a different matrix when the product genuinely needs one.

**Applies to:** every project. Mobile-specific ergonomics live in [Responsive Design](responsive-design.md).

---

## Mandatory

### Default supported matrix

Unless the project addendum records a different matrix (see below), support:

| Browser | Minimum version | Notes |
|---|---|---|
| Chrome / Edge (Chromium) | Last 2 major versions | Auto-updating — "last 2" tracks itself |
| Firefox | Last 2 major versions | |
| Safari (macOS) | Last 2 major versions | |
| Safari (iOS) | Last 2 major versions | iOS Safari is the effective engine for **all** iOS browsers (WebKit mandated by Apple) — testing Safari covers Chrome-on-iOS too |
| Samsung Internet | Latest | Meaningful Android share in some markets — check project analytics before dropping |

- **No support for Internet Explorer, any browser**, unless a project addendum explicitly documents a legacy-enterprise requirement with an ADR and a sunset date.
- Express the matrix in tooling, not just docs — `.browserslistrc` / `browserslist` field drives Autoprefixer, Babel/SWC targets, and `next/font` fallbacks automatically:
  ```
  # .browserslistrc
  > 0.5%
  last 2 versions
  Firefox ESR
  not dead
  not IE 11
  ```
- **CI runs cross-browser E2E** (Playwright covers Chromium, Firefox, and WebKit in one config) on the primary user flow at minimum — not just Chromium.
  ```ts
  // playwright.config.ts
  projects: [
    { name: 'chromium', use: devices['Desktop Chrome'] },
    { name: 'firefox', use: devices['Desktop Firefox'] },
    { name: 'webkit', use: devices['Desktop Safari'] },
  ]
  ```

### Graceful degradation, not universal support

- A feature outside the supported matrix should **degrade, not break**: feature-detect (`if ('IntersectionObserver' in window)`), never user-agent sniff.
- Core content and the primary user loop must be reachable even where an enhancement (animation, some interactive widget) isn't available. Progressive enhancement over graceful-degradation-as-afterthought.
- Polyfills are scoped and conditional (`core-js` targeted entries, or a `Module`/`nomodule` split) — never a blanket polyfill bundle shipped to every browser regardless of need.

### Recording an exception

If a project needs a different matrix (e.g. must support a specific older WebView for an embedded partner integration, or can drop mobile Safari entirely for an internal tool):

1. Record it in the project addendum's stack-notes field.
2. File an ADR: which browsers, why, what's out, and a revisit trigger (e.g. "revisit when partner SDK drops that WebView requirement").
3. Update `.browserslistrc` and the Playwright project list to match — don't let the doc and the tooling drift apart.

---

## Recommended (opt-in)

- **Real-device testing** (BrowserStack/Sauce Labs) for a product with meaningful mobile Safari or Samsung Internet traffic, beyond what Playwright's WebKit emulation catches.
- **Automated browser-usage analytics review** each quarter (from the project's own analytics, not assumptions) to confirm the matrix still matches real traffic before trimming or expanding it.

---

## Anti-Patterns (do not ship)

- "Support all browsers" with no explicit matrix and no CI coverage beyond Chromium.
- User-agent sniffing to branch behavior instead of feature detection.
- A blanket polyfill bundle shipped to every browser, including ones that don't need it.
- A `.browserslistrc` that doesn't match what's actually tested in CI.
- Dropping a browser from support without checking real traffic first.

---

## Quick Reference

```
✓ Default matrix: last 2 versions of Chrome/Edge/Firefox/Safari (incl. iOS) + latest Samsung Internet
✓ Matrix encoded in .browserslistrc, drives Autoprefixer/Babel/SWC targets
✓ Playwright CI covers Chromium + Firefox + WebKit on the primary flow
✓ Feature-detect, degrade gracefully · scoped/conditional polyfills only
✓ Different matrix needed → project addendum + ADR + revisit trigger
✗ No IE support without an explicit, ADR'd, sunset-dated exception
✗ No user-agent sniffing · no untested "works everywhere" claims
```

---
*Section version: 0.1 — initial draft*
