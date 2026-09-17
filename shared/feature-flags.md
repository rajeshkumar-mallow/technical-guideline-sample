# Feature Flags

**Read this first.** Feature flags decouple deploy from release and let risky changes ship dark. Undisciplined flags rot into permanent branches nobody trusts — this section keeps them a tool, not tech debt.

**Applies to:** Any project gating behavior behind a runtime toggle, whether via a vendor SDK or a homegrown config table.

---

## Mandatory

### Every flag has an owner and an expiry
A flag with no owner never gets removed. Track, at minimum, in the flag tool or a lightweight register (`docs/FEATURE_FLAGS.md` if no vendor dashboard):

| Field | Example |
|-------|---------|
| Key | `checkout-new-payment-form` |
| Owner | `@team-payments` |
| Type | Release / Experiment / Ops kill-switch / Permission |
| Created | 2026-08-01 |
| Revisit by | 2026-10-01 (or "on 100% rollout + 2 weeks stable") |
| Removal PR | linked once cleaned up |

### Flag types dictate lifetime
- **Release flags** (hide unfinished work) — short-lived, removed within one sprint of 100% rollout.
- **Ops kill-switches** (disable a risky integration under load) — long-lived by design, but still owned and tested.
- **Experiment flags** (A/B) — lifetime bounded by the experiment's stat-sig window; results feed a decision, then the flag is removed either way.
- **Permission/entitlement flags** (plan-gated features) — long-lived; these are product config, not tech debt, and don't need removal.

### Evaluate flags close to the render, not scattered
```ts
// One evaluation point per flag per request/session — not re-checked in ten components
const flags = await getFlags(userContext)

function CheckoutForm() {
  return flags.checkoutNewPaymentForm ? <NewPaymentForm /> : <LegacyPaymentForm />
}
```
Avoid `if (flag) { ... } else { ... }` branching duplicated across many files — centralize the branch at the boundary (route, top-level component) so removal is a small diff.

### Server-evaluated flags never leak future-feature names to the client
For unreleased/competitive-sensitive features, evaluate server-side (RSC, API response) rather than shipping the flag key + all variant code to the client bundle, which anyone can inspect. See [`../react-next/security.md`](../react-next/security.md).

### Kill-switches are tested
An ops kill-switch that's never been flipped in staging is unverified. Exercise the "off" path in CI at least once per quarter for critical switches.

## Recommended (opt-in)

- **Tooling:** a vendor (LaunchDarkly, Flagsmith, Unleash, GrowthBook) once you need targeting rules, gradual rollout percentages, or non-engineer flag management. A simple typed config object (`config/flags.ts` + environment override) is enough for a small project — don't adopt a vendor SDK for two boolean toggles.
- **Typed flag access** — generate/hand-write a typed interface (`Flags.checkoutNewPaymentForm: boolean`) instead of stringly-typed `getFlag("checkout-new-payment-form")` calls scattered through the codebase, so a typo fails at compile time, not in production.
- **Default-safe values** — if the flag service is unreachable, fall back to the safer variant (usually "off" for new features, "on" for kill-switches defaulting to the safe path). Document the fallback per flag if it's not obvious.

## Anti-Patterns (do not ship)

- Flags with no owner, no expiry, and no removal ticket — "temporary" flags that outlive the feature they gated by years.
- Nesting flags inside flags (`if (flagA) { if (flagB) { ... } }`) — creates an untested combinatorial matrix.
- Using a flag to hide a bug instead of fixing it ("disable it for now" as a permanent state).
- Client-bundling variant code for unreleased features gated only by a client-visible boolean — the code (and the fact the feature exists) ships to every user regardless of flag state.
- Checking the same flag in a dozen unrelated files instead of centralizing the branch.

## Quick Reference
```
✓ Every flag: owner, type, revisit-by date
✓ Release flags removed within one sprint of full rollout
✓ Evaluate once, branch at the boundary — not scattered checks
✓ Unreleased-feature flags evaluated server-side, not client-visible
✓ Kill-switches exercised in CI/staging periodically
✗ No owner-less or expiry-less flags
✗ No nested flag combinations
✗ No flags used to permanently hide unfixed bugs
```

---
*Section version: 0.1 — initial draft*
