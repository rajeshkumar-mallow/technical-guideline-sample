# API Integration

**Read this first.** Next.js gives you two server-side entry points — Route Handlers and Server Actions — and picking the wrong one for the job creates inconsistent error handling and unnecessary client complexity. This file governs when to use which, and the error contract both must follow.

**Applies to:** All server-side request handling in a React/Next 15 App Router project.

---

## Mandatory

### Route Handlers vs. Server Actions — pick by caller, not habit

| Use **Route Handler** (`app/api/.../route.ts`) when | Use **Server Action** when |
|---|---|
| Called by a non-React client (mobile app, webhook, third-party) | Called only from your own React components |
| Needs a stable public URL / REST or JSON contract | Form submission or a direct mutation from UI |
| Response isn't HTML/RSC (file download, webhook ack) | Progressive enhancement matters (works without JS via `<form action>`) |

Don't build a Route Handler just to `fetch()` it from your own client component — call a Server Action directly instead and skip the network hop.

### Structured error contract
Every Route Handler that returns JSON, and every Server Action that reports failure, uses a stable error shape — mirrors the convention in the Rails guideline:

```ts
// lib/api-error.ts
export type ApiError = {
  error: {
    code: string;        // stable, machine-readable: "validation_failed", "not_found"
    message: string;     // human-readable summary, safe to show
    details?: unknown;   // structured, e.g. field errors
  };
};
```

```ts
// app/api/orders/route.ts
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const body = await request.json();
  const parsed = orderSchema.safeParse(body);
  if (!parsed.success) {
    return NextResponse.json(
      { error: { code: "validation_failed", message: "Invalid order payload", details: parsed.error.flatten() } },
      { status: 422 },
    );
  }
  try {
    const order = await placeOrder(parsed.data);
    return NextResponse.json({ order }, { status: 201 });
  } catch (err) {
    return NextResponse.json(
      { error: { code: "internal_error", message: "Could not place order" } },
      { status: 500 },
    );
  }
}
```

Never let an unhandled exception in a Route Handler reach the client as a raw stack trace — catch, classify, and return the structured shape.

### Server Actions apply resilience patterns too
Any Server Action or Route Handler that calls an external provider (payments, email, storage) follows the same resilience rules as client-side fetches in [frontend/api-resilience.md](../frontend/api-resilience.md): explicit timeouts, retry only on transient failures, idempotency keys on mutations that must not duplicate.

```ts
"use server";
export async function chargeCard(input: ChargeInput) {
  const parsed = chargeSchema.safeParse(input);
  if (!parsed.success) return { error: { code: "validation_failed", message: "Invalid payment details" } };

  try {
    const charge = await stripeClient.charges.create(parsed.data, {
      idempotencyKey: input.idempotencyKey,
    });
    return { charge };
  } catch (err) {
    return { error: classifyProviderError(err) }; // transient vs. permanent, see frontend/api-resilience.md
  }
}
```

### Auth check on every entry point
Route Handlers and Server Actions are **not automatically protected by page-level auth** — a page's middleware guard doesn't cover a Server Action invoked from elsewhere. Re-check the session/permissions inside the action or handler itself (see [react-next/auth.md](auth.md)).

---

## Recommended (opt-in)

- **Zod-typed Route Handlers** via a thin wrapper that parses the body/query and returns the structured error automatically, so every handler doesn't hand-roll the same try/catch.
- **Revalidation helpers** (`revalidatePath`/`revalidateTag`) called from the Server Action right after a successful mutation, not from the client, so cache invalidation can't be skipped by a client that forgets to call it.
- **Versioned API routes** (`app/api/v1/...`) once the Route Handler surface is consumed by external clients you don't control — record the versioning policy in an ADR.

---

## Anti-Patterns (do not ship)

- A Route Handler that exists only to be called by your own client component — use a Server Action.
- Returning inconsistent error shapes across handlers (some `{ error: "..." }` strings, others structured objects).
- Trusting that a Server Action is protected because the page that renders its form is behind auth middleware — actions are directly callable.
- Swallowing provider errors without classification, retrying a permanent failure (e.g. 422) as if it were transient.
- Passing full domain objects across the client/server boundary as action arguments when only an ID is needed — keep payloads minimal and re-fetch/re-authorize server-side.

## Quick Reference

```
✓ Route Handler for external/non-React callers; Server Action for your own UI
✓ Structured { error: { code, message, details } } on every failure path
✓ Auth re-checked inside every Server Action / Route Handler
✓ Provider calls: timeout, classify, retry-if-transient, idempotency key
✗ Route Handler that only your own client calls
✗ Inconsistent or raw error shapes
✗ Assuming page-level auth protects a Server Action
```

---
*Section version: 0.1 — initial draft*
