# Security

**Read this first.** Client-side code runs in an environment you don't control — the browser, the user's extensions, and anything injected between them. These rules exist to keep a compromised dependency, a reflected input, or a misconfigured header from turning into a working exploit.

**Applies to:** any frontend codebase (framework-agnostic). Next.js-specific additions live in [`next/security.md`](next/security.md).

---

## Mandatory

### XSS prevention
- **Never** assign untrusted strings to `innerHTML`, `outerHTML`, `document.write`, or a framework's raw-HTML escape hatch (`dangerouslySetInnerHTML`, Vue's `v-html`) without passing them through a sanitizer first.
- When rich text truly must be rendered (CMS content, user bios with formatting), sanitize server-side **and** client-side with an allowlist-based sanitizer (e.g. `DOMPurify`), never a denylist/regex strip.
- Default to **plain text** rendering (`textContent`, JSX `{}` interpolation) unless rich text is an explicit product requirement.

```ts
import DOMPurify from "dompurify";

function renderUserBio(rawHtml: string): string {
  return DOMPurify.sanitize(rawHtml, { ALLOWED_TAGS: ["b", "i", "a", "p"] });
}
```

### Content Security Policy (CSP)
- Ship a CSP header (or `<meta http-equiv="Content-Security-Policy">` as a fallback) restricting `script-src` to self + explicitly allowlisted origins.
- Use **nonces** for any inline `<script>` that can't be avoided — never `unsafe-inline` in production.
- Set `object-src 'none'` and `base-uri 'self'` to close two common injection vectors.
- Report violations to an endpoint (`report-uri` / `report-to`) so drift is visible before it's exploited.

### Dependency scanning
- Run `npm audit` (or `pnpm audit` / `yarn audit`) in CI on every PR; block merge on new **high/critical** advisories.
- Use Dependabot/Renovate or Snyk for automated PRs on vulnerable transitive dependencies — see [Approved Libraries](../shared/approved-libraries.md) for the broader dependency-hygiene process.
- Pin lockfiles (`package-lock.json` / `pnpm-lock.yaml`) and commit them — never `npm install` without a lockfile in CI.

### Secrets never reach the client bundle
- Anything bundled into client JS is **public**, full stop — treat build-time env injection (`NEXT_PUBLIC_*`, `VITE_*`, `REACT_APP_*`) as a publish action, not configuration.
- API keys for privileged operations (payments, admin APIs, anything with a write scope) stay server-side; the client calls your backend, which holds the real credential.
- Grep the built bundle for known secret patterns as a CI gate before first deploy and periodically after (ties to [Environment & Configuration](../shared/environment-config.md)).

### Other baseline headers
| Header | Purpose |
|---|---|
| `X-Content-Type-Options: nosniff` | Stops MIME-sniffing attacks |
| `X-Frame-Options: DENY` (or CSP `frame-ancestors`) | Clickjacking protection |
| `Referrer-Policy: strict-origin-when-cross-origin` | Limits referrer leakage |
| `Permissions-Policy` | Explicitly disables unused browser features (camera, geolocation, etc.) |

## Recommended (opt-in)

- **Subresource Integrity (SRI)** hashes on any third-party `<script>`/`<link>` loaded from a CDN outside your build pipeline.
- **Trusted Types** (Chromium-supported) to make DOM-XSS sinks a build-time error rather than a runtime risk, once your framework/libraries support it.
- Automated dynamic scanning (OWASP ZAP baseline scan) against staging as a periodic CI job for higher-risk applications — record the decision to enable this via ADR since it adds pipeline time.

## Anti-Patterns (do not ship)

- Building HTML via string concatenation/template literals and injecting it with `innerHTML`.
- `unsafe-inline` or `unsafe-eval` in a production CSP without a documented, time-boxed ADR.
- Client-side "security" that only hides a UI element — the real check must happen server-side (auth, entitlements, feature gating).
- Committing `.env` files with real credentials, or embedding a private API key in client-bundled code "temporarily."
- Ignoring `npm audit` output by pinning to `--force` without triaging the advisory.

## Quick Reference

```
✓ Sanitize (DOMPurify) any rendered rich text — never raw innerHTML
✓ CSP with nonces, no unsafe-inline, object-src 'none'
✓ npm/pnpm/yarn audit gated in CI, lockfile committed
✓ Only NEXT_PUBLIC_/VITE_/REACT_APP_-prefixed vars in client bundle — never real secrets
✓ Security headers: X-Content-Type-Options, X-Frame-Options, Referrer-Policy, Permissions-Policy
✗ innerHTML / dangerouslySetInnerHTML with unsanitized input
✗ Secrets baked into client JS
✗ UI-only authorization
```

---
*Section version: 0.1 — initial draft*
