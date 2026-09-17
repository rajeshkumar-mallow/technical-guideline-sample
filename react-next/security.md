# Security (React/Next)

**Read this first.** Server Components and Server Actions blur the client/server line in ways that create Next-specific footguns — an object that "just works" as a prop can silently leak fields to the browser, and a Server Action looks like a plain function call but is a public HTTP endpoint. This file covers what's specific to React/Next; general web security (XSS, CSP headers, dependency scanning) lives in [frontend/security.md](../frontend/security.md).

**Applies to:** all React/Next projects.

---

## Mandatory

### Server Actions are public endpoints
A Server Action (`"use server"`) is callable by anyone who can reach your app, regardless of which UI calls it — treat every argument as untrusted input, exactly like a REST/JSON API body.

```ts
"use server";
import { z } from "zod";

const UpdateProfileSchema = z.object({
  name: z.string().min(1).max(120),
  bio: z.string().max(500).optional(),
});

export async function updateProfile(formData: FormData) {
  const session = await getSession();
  if (!session) throw new Error("Unauthorized");

  const parsed = UpdateProfileSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { error: parsed.error.flatten() };

  await db.user.update({ where: { id: session.userId }, data: parsed.data });
}
```

- **Validate every Server Action input server-side** (Zod or equivalent) — client-side validation is UX only, never a security boundary. See [Forms & Validation](forms-validation.md).
- **Re-check authorization inside the Server Action itself** — do not rely on the calling component being behind an authenticated route; the action is reachable directly.
- Server Actions mutating data must be **idempotent-safe or explicitly guarded** against duplicate submission (double-click, retry) the same way any mutating endpoint would be.

### `NEXT_PUBLIC_` is public, permanently
Any env var prefixed `NEXT_PUBLIC_` is inlined into the client bundle at build time and is visible to anyone who views source. Never put a secret, API key, or internal URL with sensitive query params behind that prefix.

```
# .env — correct
DATABASE_URL=postgres://...        # server-only, never NEXT_PUBLIC_
STRIPE_SECRET_KEY=sk_live_...      # server-only

NEXT_PUBLIC_ANALYTICS_ID=G-XXXX    # fine — meant to be public
```

Run a grep for `NEXT_PUBLIC_` against your secrets list before every release; a leaked secret here isn't rotatable by revoking a deploy — it's already cached in every browser and CDN edge that fetched the bundle.

### Server-to-Client data exposure
Props passed from a Server Component to a Client Component are serialized and sent to the browser. **Never spread a full database record as props** — select only the fields the client actually needs.

```tsx
// Anti-pattern — leaks passwordHash, internalNotes, every column
<ClientProfileCard user={dbUser} />

// Correct — explicit, minimal shape
<ClientProfileCard user={{ id: dbUser.id, name: dbUser.name, avatarUrl: dbUser.avatarUrl }} />
```

Apply the same field-level discipline described in the Rails guideline's presenter/serializer pattern — a dedicated mapping function (`toProfileCardProps(user)`) is the single place that decides what's safe to send to the client, not ad-hoc destructuring at each call site.

### CSP with streaming SSR
Because the App Router streams HTML progressively, a static CSP `<meta>` tag is insufficient for inline scripts added by the framework. Use **nonce-based CSP** set via `middleware.ts`, generating a fresh nonce per request and passing it through to any inline script:

```ts
// middleware.ts
const nonce = crypto.randomUUID();
const csp = `script-src 'self' 'nonce-${nonce}'; object-src 'none'; base-uri 'self';`;
response.headers.set("Content-Security-Policy", csp);
response.headers.set("x-nonce", nonce);
```

Never use `'unsafe-inline'` as a substitute for wiring the nonce through — it defeats the point of CSP.

---

## Recommended (opt-in)

- Add a lint rule or code-review checklist item flagging any Server Component prop spread (`{...record}`) passed into a component imported from a file with `"use client"`.
- Rate-limit sensitive Server Actions (auth, payment-adjacent, expensive mutations) the same way you would a REST endpoint — see [API Resilience](../frontend/api-resilience.md) for the underlying pattern.

---

## Anti-Patterns (do not ship)

- Trusting a Server Action's input because "the button is only shown to admins" — the action itself has no idea who called it unless it checks.
- Any secret, API key, or internal-only URL behind `NEXT_PUBLIC_`.
- Passing a raw ORM/DB record directly as a Client Component prop.
- `'unsafe-inline'` in CSP as a shortcut instead of nonce wiring.
- Server Actions with no re-validation of authorization, relying solely on the page being behind auth middleware.

---

## Quick Reference

```
✓ Every Server Action validates input server-side (Zod) and re-checks auth
✓ NEXT_PUBLIC_ only for values safe to be fully public, forever
✓ Explicit minimal prop shape from Server → Client Component, never a raw record spread
✓ Nonce-based CSP via middleware for streaming SSR
✗ No trusting client-side validation as a security boundary
✗ No secrets behind NEXT_PUBLIC_, no 'unsafe-inline' CSP shortcuts
```

See also: [frontend/security.md](../frontend/security.md), [Forms & Validation](forms-validation.md), [API Integration](api-integration.md), [Auth](auth.md).

---
*Section version: 0.1 — initial draft*
