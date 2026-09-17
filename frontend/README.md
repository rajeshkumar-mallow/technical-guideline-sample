# Frontend Guidelines

Framework-agnostic frontend rules — applies to any frontend stack (React, Next.js, or otherwise). Assumes [`../shared/`](../shared/README.md) as a baseline. If the project uses React/Next, also see [`../react-next/`](../react-next/README.md) for framework-specific rules that extend these.

| File | Covers |
|------|--------|
| [HTML & CSS](html-css.md) | Semantic HTML, CSS methodology/naming |
| [Styling Foundations](styling-foundations.md) | CSS reset, preprocessor choice, design tokens, UI library selection |
| [Design to Code](design-to-code.md) | Figma token sync, component handoff, style guide/Storybook |
| [JavaScript](javascript.md) | Language conventions, module patterns, async/error handling |
| [Accessibility](accessibility.md) | WCAG target, ARIA, keyboard/focus, reduced-motion, axe-core |
| [Performance](performance.md) | Bundle size, lazy loading, Core Web Vitals, images/fonts |
| [Security](security.md) | XSS, CSP, sanitization, dependency scanning |
| [Authentication](auth.md) | Token storage, CSRF, protected routes |
| [PWA & Service Workers](pwa-service-worker.md) | Caching, offline support, client-side storage |
| [Realtime & WebSockets](realtime-websockets.md) | WebSocket/SSE patterns, reconnection, handshake auth |
| [Internationalization](i18n.md) | Locale structure, pluralization, RTL, formatting |
| [Forms & Validation](forms-validation.md) | Native form handling, accessible errors, validation patterns |
| [Error Monitoring](error-monitoring.md) | Client error tracking setup |
| [Responsive Design](responsive-design.md) | Mobile-first, breakpoints, touch ergonomics |
| [Tooling & Build](tooling-build.md) | Bundler/package manager, build configuration |
| [Testing Tooling](testing-tooling.md) | Unit/component/e2e tool choices |
| [Analytics & Tracking](analytics-tracking.md) | Script loading, event naming, consent |
| [SEO & Metadata](seo-metadata.md) | Meta tags, Open Graph, structured data |
| [API Resilience](api-resilience.md) | Client-side fetch timeouts, retry/backoff, error classification |

Back to [root guideline](../README.md).
