# Analytics & Tracking (Frontend)

**Read this first.** Analytics scripts are one of the most common sources of blocked main-thread time, privacy violations, and silently inconsistent data. This file sets loading, naming, and consent rules that apply to any analytics/tracking integration. Next.js-specific script-loading and route-change tracking patterns are in [`next/analytics-tracking.md`](next/analytics-tracking.md).

**Applies to:** any third-party analytics, product-analytics, or tag-manager script.

---

## Mandatory

### Consent before collection
- No analytics or tracking script fires **before consent is captured**, in any jurisdiction the product serves, unless the vendor is strictly necessary (e.g. basic uptime/error monitoring with no PII). This is not regulation-specific — it's the baseline; region-specific rules layer on top (see [`privacy-compliance.md`](../shared/privacy-compliance.md)).
- Consent state gates script injection, not just event sending — do not load the script and then silently drop events; loading the script alone can still set cookies/fingerprint.
- Provide a way to withdraw consent that actually stops future collection, not just hides a banner.

### Script loading
- Load third-party analytics **asynchronously** (`async` or `defer`) — never a blocking `<script>` in `<head>`.
- Prefer a single tag-manager entry point (e.g. one loader script) over multiple independent vendor `<script>` tags, to keep the consent gate in one place and bound the performance cost.
- Self-host or proxy through your own domain where the vendor supports it, to reduce third-party DNS/connection overhead and improve resilience to ad-blockers skewing data.

### Event naming convention
Pick **one** convention project-wide and enforce it in code review — inconsistent naming is the #1 cause of unusable analytics data.

**Baseline convention: `object_action`, snake_case** (e.g. `signup_completed`, `cart_item_added`, `checkout_started`). Record the chosen convention in the project addendum if a different one is adopted.

| Do | Don't |
|----|-------|
| `signup_completed` | `SignupComplete`, `signup-complete`, `completed_signup` |
| `video_played` | `playVideo`, `video play` |

- Event properties use consistent casing (snake_case) and consistent types across events that share a property (e.g. `user_id` is always a string, everywhere).
- Maintain a **tracking plan** (a checked-in spec, e.g. `docs/tracking-plan.md` per project) listing every event, its properties, and when it fires — treat it like an API contract, reviewed in PRs that add/change events.

### PII discipline
- Never send email, name, phone, or free-text user input as an event property unless explicitly required and approved (ties to [`privacy-compliance.md`](../shared/privacy-compliance.md)).
- Use opaque internal IDs, not raw identifiers, wherever the vendor allows.

---

## Recommended (opt-in)

- **Server-side tracking / CAPI** for critical conversion events, to reduce reliance on client-side script survival through ad-blockers — adopt when data accuracy on a key funnel matters enough to justify the added backend surface.
- **Feature-flag-gated rollout** of new tracking to validate event volume/shape before it reaches 100% of traffic.

---

## Anti-Patterns (do not ship)

- Blocking `<script>` tags for analytics in `<head>`.
- Firing tracking calls before a consent decision is recorded.
- Ad-hoc event names invented per-feature with no shared naming convention.
- Sending raw PII (email, full name) as an event property "just in case it's useful later."
- No tracking plan — events added ad-hoc with no record of what they mean.

---

## Quick Reference

```
✓ Consent gates script load, not just event send
✓ Async/deferred script loading, single tag-manager entry point where possible
✓ object_action snake_case event naming, enforced project-wide
✓ Tracking plan checked into the repo, reviewed like an API contract
✗ No blocking analytics scripts
✗ No tracking before consent
✗ No raw PII in event properties
```

---
*Section version: 0.1 — initial draft*
