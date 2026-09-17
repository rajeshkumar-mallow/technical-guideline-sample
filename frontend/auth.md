# Authentication (Frontend)

**Read this first.** The frontend never owns authentication logic — it owns how credentials are stored, attached to requests, and cleared. Get token storage wrong and every other security control (CSP, sanitization) becomes a mitigation for a hole you didn't need to open.

**Applies to:** any frontend consuming a session or token-based backend. Next.js-specific middleware and session patterns live in [`next/auth.md`](next/auth.md).

---

## Mandatory

### Token storage: httpOnly cookies over localStorage
- **Default to httpOnly, `Secure`, `SameSite=Lax` (or `Strict`) cookies** set by the server for session/auth tokens. JavaScript cannot read an httpOnly cookie, so a successful XSS injection can't exfiltrate it.
- **Do not store access/refresh tokens in `localStorage` or `sessionStorage`.** Both are fully readable by any script running on the page — one XSS bug becomes full account takeover. See [Security](security.md) for why XSS can't be fully ruled out even with mitigations in place.
- If the backend is a separate origin and cookies aren't viable (e.g. a pure SPA against a third-party API you don't control), hold the token **in memory only** (a module-level variable or in-memory store) — never persisted storage — and accept that a hard refresh requires re-auth or a silent-refresh flow via an httpOnly refresh cookie.

```ts
// Acceptable: in-memory access token, refreshed via httpOnly cookie
let accessToken: string | null = null;

export function setAccessToken(token: string) {
  accessToken = token;
}

export function getAuthHeader(): Record<string, string> {
  return accessToken ? { Authorization: `Bearer ${accessToken}` } : {};
}
```

### CSRF handling
- Cookie-based auth **must** pair with CSRF protection: `SameSite=Lax` cookies block most cross-site cases, but state-changing requests (`POST`/`PUT`/`PATCH`/`DELETE`) still need a CSRF token (double-submit cookie or synchronizer token) validated server-side.
- Read the CSRF token from a non-httpOnly cookie or a meta tag injected by the server, and attach it as a header (`X-CSRF-Token`) on every mutating request.

### Protected routes
- Route guards are a **UX convenience**, not a security boundary — the real enforcement is server-side (session validation, authorization checks on the API). A client-side redirect from `/dashboard` to `/login` only prevents a confusing flash of content; it does not protect data.
- Fetch the current user/session status once at app boot (or via a framework-level loader) and gate rendering on it, rather than checking auth ad hoc in every component.

### Sign-out
- Sign-out must clear both the client-held token (memory) and invalidate the server-side session/cookie — clearing only the client side leaves a valid session usable by anyone who has the cookie.
- Expose a **"sign out everywhere"** action (revoke all sessions) when the product has account-security requirements (shared devices, high-value accounts).

## Recommended (opt-in)

- **Silent token refresh** via a short-lived access token + long-lived httpOnly refresh cookie, refreshed transparently before expiry.
- **Device/session list UI** showing active sessions with per-session revoke, for products where account takeover risk is elevated.

## Anti-Patterns (do not ship)

- Storing a JWT or session token in `localStorage`/`sessionStorage` "because it's simpler."
- Relying solely on hiding a nav link or route to protect sensitive data — the API endpoint behind it must independently authorize.
- Sending the CSRF token value in a cookie **and** trusting the cookie alone (defeats double-submit — the header/body value must be compared server-side against the cookie).
- Skipping sign-out session invalidation server-side (token just discarded client-side).

## Quick Reference

```
✓ httpOnly + Secure + SameSite cookies for session tokens
✓ In-memory only if cookies aren't viable — never localStorage/sessionStorage
✓ CSRF token on every mutating request when using cookie auth
✓ Server-side enforcement backs every client-side route guard
✓ Sign-out clears client state AND invalidates server session
✗ Tokens in localStorage/sessionStorage
✗ Route guards as the only access control
```

---
*Section version: 0.1 — initial draft*
