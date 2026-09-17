# Forms & Validation (React/Next)

**Read this first.** Forms are where user input meets your data layer — the point most bugs and most security holes come from. This file governs how forms are built, validated, and submitted in React/Next projects, extending the generic patterns in [frontend/forms-validation.md](../frontend/forms-validation.md).

**Applies to:** Every form in a React/Next 15 App Router project, client-rendered or Server-Action-backed.

---

## Mandatory

### Validate on both client and server
Client-side validation is UX only. **Never trust it as the source of truth.** Every Server Action and Route Handler re-validates input with the same schema, because a request can always bypass the browser.

```ts
// lib/schemas/signup.ts
import { z } from "zod";

export const signupSchema = z.object({
  email: z.string().email(),
  password: z.string().min(12),
  displayName: z.string().min(1).max(80),
});

export type SignupInput = z.infer<typeof signupSchema>;
```

### `react-hook-form` + `zod` for client-rendered forms
This is the required pairing for any interactive, client-side form (multi-step, dynamic fields, inline validation feedback). Use `@hookform/resolvers/zod` to share the schema between client and server.

```tsx
"use client";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { signupSchema, type SignupInput } from "@/lib/schemas/signup";

export function SignupForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } =
    useForm<SignupInput>({ resolver: zodResolver(signupSchema) });

  async function onSubmit(data: SignupInput) {
    const result = await signupAction(data);
    if (result?.error) { /* surface field/root errors */ }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <input {...register("email")} aria-invalid={!!errors.email} />
      {errors.email && <p role="alert">{errors.email.message}</p>}
      <button disabled={isSubmitting}>Sign up</button>
    </form>
  );
}
```

### Server Actions with `useActionState` for progressive enhancement
Prefer a plain `<form action={...}>` bound to a Server Action with `useActionState` when the form doesn't need rich client interactivity. This works **without JavaScript** — the form still submits and re-renders with errors if the client bundle fails to load or hasn't hydrated yet.

```tsx
// app/signup/actions.ts
"use server";
import { signupSchema } from "@/lib/schemas/signup";

export async function signupAction(_prev: unknown, formData: FormData) {
  const parsed = signupSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) {
    return { error: parsed.error.flatten().fieldErrors };
  }
  // create the user...
  return { error: null };
}
```

```tsx
// app/signup/page.tsx
"use client";
import { useActionState } from "react";
import { signupAction } from "./actions";

export default function SignupPage() {
  const [state, formAction, pending] = useActionState(signupAction, { error: null });
  return (
    <form action={formAction}>
      <input name="email" type="email" required />
      {state.error?.email && <p role="alert">{state.error.email[0]}</p>}
      <button disabled={pending}>Sign up</button>
    </form>
  );
}
```

### Accessible error handling
- Errors are associated with their field via `aria-describedby`, and announced via `role="alert"` on first appearance (see [frontend/accessibility.md](../frontend/accessibility.md)).
- Never rely on color alone to indicate an invalid field.
- Preserve entered values on validation failure — never clear the form.

---

## Recommended (opt-in)

- **Multi-step wizards:** keep step state in the URL (search params) or `useActionState`-driven server state rather than client-only state, so a refresh doesn't lose progress. Record the chosen approach in an ADR if it diverges from a single-page form.
- **Optimistic UI** via `useOptimistic` for high-frequency, low-risk mutations (e.g. a like button) — not for anything that can fail validation server-side.
- **Field-level async validation** (e.g. "email already taken") debounced and called via a Server Action, not a Route Handler, to reuse the same schema.

---

## Anti-Patterns (do not ship)

- Trusting `react-hook-form`'s client validation as the only check — always re-validate in the Server Action/Route Handler.
- Building fully client-controlled forms (`useState` per field + manual `fetch`) when `react-hook-form` or a Server Action would do — don't hand-roll what the pairing already solves.
- Disabling the submit button as the *only* duplicate-submission guard — also guard server-side with idempotency where the action isn't naturally idempotent (see [frontend/api-resilience.md](../frontend/api-resilience.md)).
- Returning raw Zod error objects to the client — flatten them into a stable, presentable shape.
- Skipping `noValidate` + custom messaging and relying on inconsistent native browser validation UI across browsers.

## Quick Reference

```
✓ Shared zod schema between client and server
✓ react-hook-form + zodResolver for interactive forms
✓ useActionState + Server Actions for progressive enhancement
✓ Server-side re-validation on every submission path
✓ Accessible, field-associated error messages
✗ Client-only validation
✗ Hand-rolled form state when RHF/Server Actions fit
✗ Clearing form values on validation error
```

---
*Section version: 0.1 — initial draft*
