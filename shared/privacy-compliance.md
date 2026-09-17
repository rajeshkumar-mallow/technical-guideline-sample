# Privacy & Data Compliance

**Read this first.** Strict data-minimization and consent hygiene are mandatory baseline for every project — regulation-specific obligations (GDPR, CCPA, HIPAA, etc.) are **not** assumed and are handled as a Compliance add-on once legal assigns them, mirroring how the backend guideline scopes privacy.

**Applies to:** Any client-side code that stores, transmits, or displays user data — forms, analytics, storage, third-party embeds.

---

## Mandatory

### Data minimization on the client
- Don't collect a field in a form, don't request a scope from an OAuth provider, and don't mirror a backend field into client state unless it's used on that screen. Every field collected needs a reason a reviewer can state in one sentence.
- Don't log full request/response bodies containing user data to the browser console or to a client-side error monitor (see [`../frontend/error-monitoring.md`](../frontend/error-monitoring.md)) in production builds.

### Cookie / storage consent
- **No non-essential cookies, analytics, or tracking scripts fire before consent** is given, for any project operating in a jurisdiction requiring prior consent (EU/UK at minimum — treat as default-on unless a project addendum says otherwise).
- Consent categories, minimum: `essential` (always on, e.g. session/CSRF cookie), `analytics`, `marketing`. A consent banner gates categories independently — accepting "essential" must not silently enable analytics.
- Store the consent decision itself (category + timestamp + policy version) — this record is required to prove compliance later.
- Wire consent state into [`../frontend/analytics-tracking.md`](../frontend/analytics-tracking.md) — the analytics loader checks consent before injecting any script tag, not after.

```ts
// Analytics scripts only load after explicit opt-in
function loadAnalytics(consent: ConsentState) {
  if (!consent.analytics) return
  injectScript('https://analytics.example.com/script.js')
}
```

### What's safe vs. unsafe in client-side storage
| Storage | Safe to put here | Never put here |
|---|---|---|
| `localStorage` / `sessionStorage` | UI preferences, non-sensitive cached view state, feature-flag cache | Passwords, full auth tokens (see [`../frontend/auth.md`](../frontend/auth.md)), SSNs/government IDs, full card numbers, raw API responses containing PII |
| Cookies (non-HttpOnly) | Consent state, locale, non-sensitive UI state | Session tokens (use `HttpOnly`), anything readable by injected JS |
| IndexedDB (PWA offline cache) | App data the user already has access to, explicitly needed offline | Data for users other than the current one; sensitive fields without a retention/eviction policy |

`localStorage` is plaintext, unencrypted, and readable by any script on the page (including a successful XSS payload) — treat it as public-to-the-tab, never as secure storage.

### Third-party scripts are inventoried
Every third-party script/embed (chat widgets, ad pixels, session-replay tools) is listed with what data it can access and whether it's consent-gated. Session-replay/heatmap tools in particular can capture form input — mask sensitive fields (`data-privacy="mask"` or equivalent) by default.

## Recommended (opt-in)

- **Consent Management Platform (CMP)** (OneTrust, Cookiebot, or a lightweight in-house banner) once the project needs jurisdiction-aware consent (e.g., different rules for EU vs. US visitors) rather than a single global banner.
- **Do Not Track / Global Privacy Control** honoring — respect the GPC signal as an analytics opt-out where the jurisdiction requires it.

### Compliance add-on
When legal assigns a formal obligation, enable the matching pack and record it in the project addendum — same pattern as the backend guideline's Compliance packs:

| Pack | Typical additions |
|------|---|
| **GDPR** | Cookie banner with granular categories, DSR (data subject request) export/delete flow triggers, data-processing record |
| **CCPA/CPRA** | "Do Not Sell/Share My Info" link, opt-out signal honoring |
| **HIPAA** | No PHI in client analytics/error monitors under any circumstance; session timeout on idle |

## Anti-Patterns (do not ship)

- Firing analytics/marketing scripts on page load before consent is captured.
- Storing auth tokens, passwords, or government ID numbers in `localStorage`/`sessionStorage`.
- Sending full form payloads (including sensitive fields) to an error-monitoring service on every submit failure.
- A single "Accept All" cookie banner with no granular category control, in a jurisdiction requiring it.
- Session-replay tools capturing unmasked password/payment fields.

## Quick Reference
```
✓ Minimize fields collected; state the reason for each
✓ No non-essential cookies/analytics before consent
✓ Consent decision itself is recorded (category + timestamp + version)
✓ Sensitive data never in localStorage/sessionStorage
✓ Third-party scripts inventoried; session-replay masks sensitive fields
✓ Compliance add-on enabled only when legal assigns an obligation
✗ No PII/tokens in client-side storage
✗ No tracking before explicit consent
```

---
*Section version: 0.1 — initial draft*
