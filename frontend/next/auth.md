# Authentication (React/Next)

**Read this first.** Auth state has to be correctly readable from three different execution contexts — Server Components, Client Components, and Server Actions/Route Handlers — each with a different API for getting at the same session. Getting this wrong produces either a flash-of-unauthenticated-content bug or a real auth bypass.

**Applies to:** all React/Next projects. General token-storage and CSRF principles live in [frontend/auth.md](../auth.md); this file covers what's Next-specific.

---

## Mandatory

### Baseline: Auth.js (NextAuth) with cookie-based sessions
Default to **Auth.js** (formerly NextAuth.js) for session management unless a project has a specific reason not to (e.g. an existing enterprise SSO/IdP integration with its own SDK — record that choice as an ADR). Rationale: it's the framework-native option with first-class App Router support (Route Handlers, middleware helpers, Server Component session reads), handles CSRF and secure cookie flags correctly by default, and avoids hand-rolling session/JWT logic.

- Sessions are **httpOnly, secure, sameSite cookies** — never store the session token in `localStorage` or a client-readable cookie. This is non-negotiable regardless of auth library.
- Database sessions (not just signed JWTs) when the product needs server-side revocation ("sign out everywhere") — see the equivalent requirement in the Rails guideline's §4.

### Middleware for route protection
Use `middleware.ts` to gate protected route groups before any page code runs — this is cheaper and more consistent than per-page auth checks, and prevents the "forgot to add the check" class of bug.

```ts
// middleware.ts
export async function middleware(request: NextRequest) {
  const session = await getSessionFromCookie(request);
  if (!session && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
}

export const config = { matcher: ["/dashboard/:path*"] };
```

Middleware is a **first line of defense for UX (redirect unauthenticated users early)**, not the only authorization check — every Server Action and Route Handler still re-validates the session itself (§ [Security](security.md)), since middleware can be bypassed by calling an endpoint directly.

### Reading session per context

| Context | How to read the session |
|---|---|
| **Server Component** | `await auth()` (Auth.js) or equivalent server-side session read — never pass the session down as a prop through many layers; re-read it at the point of use or via a request-scoped cache |
| **Client Component** | `useSession()` hook from a `SessionProvider` wrapping the app — never assume a server-read session value is still valid client-side, it can be stale |
| **Server Action / Route Handler** | `await auth()` inside the action/handler itself — **always**, not inferred from which page called it (see [Security](security.md)) |

```tsx
// Server Component
export default async function DashboardPage() {
  const session = await auth();
  if (!session) redirect("/login");
  return <Dashboard user={session.user} />;
}
```

### Sign-out everywhere
When the product has account-security requirements, implement session revocation (database session store + a "revoke all sessions" action), not just clearing the current browser's cookie — the same requirement as the Rails guideline's §4.

---

## Recommended (opt-in)

- OAuth/SSO providers (Google, Microsoft, SAML), MFA, and magic links are **add-ons** on top of the Auth.js baseline — enable per project need, record via ADR, and adding a provider should be a config change to the `Auth.js` provider list, not a rewrite of the session model.
- For API-token/machine-to-machine auth (webhooks, third-party integrations calling your Next app), use a separate, explicitly-scoped token mechanism — do not reuse the user session cookie scheme.

---

## Anti-Patterns (do not ship)

- Session token in `localStorage`, `sessionStorage`, or a non-`httpOnly` cookie.
- A Server Action or Route Handler that trusts the page-level middleware check instead of re-validating the session itself.
- Passing the session object as a prop through many component layers instead of reading it where needed.
- Client Component auth checks (`useSession()`) used as the *only* gate on sensitive UI — always paired with a server-side check, since client-side checks can be bypassed by disabling JS or calling APIs directly.
- Hand-rolled JWT session logic without a documented reason to avoid Auth.js.

---

## Quick Reference

```
✓ Auth.js (NextAuth) baseline; httpOnly/secure/sameSite cookie sessions
✓ middleware.ts gates protected routes for UX; every Server Action/Route Handler still re-checks auth
✓ auth() in Server Components/Actions; useSession() in Client Components
✓ Database-backed sessions when "sign out everywhere" is required
✗ No session tokens in localStorage or client-readable cookies
✗ No Server Action trusting the caller's route to have already checked auth
```

See also: [frontend/auth.md](../auth.md), [Security](security.md), [Middleware-related routing notes](routing-data-fetching.md).

---
*Section version: 0.1 — initial draft*
