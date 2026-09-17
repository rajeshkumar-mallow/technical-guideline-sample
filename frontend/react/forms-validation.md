# Forms & Validation (React)

**Read this first.** Forms are where user input meets your data layer — the point most bugs and most security holes come from. This file governs client-rendered React form patterns, extending the generic baseline in [`../forms-validation.md`](../forms-validation.md). Server Action-specific submission handling (Next.js) is in [`../next/forms-validation.md`](../next/forms-validation.md).

**Applies to:** any React codebase.

---

## Mandatory

### Validate on both client and server
Client-side validation is UX only. **Never trust it as the source of truth.** Every server-side handler (a REST endpoint, a Server Action, whatever the backend boundary is) re-validates input with the same schema, because a request can always bypass the browser.

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
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { signupSchema, type SignupInput } from "@/lib/schemas/signup";

export function SignupForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } =
    useForm<SignupInput>({ resolver: zodResolver(signupSchema) });

  async function onSubmit(data: SignupInput) {
    const result = await submitSignup(data);
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

### Accessible error handling
- Errors are associated with their field via `aria-describedby`, and announced via `role="alert"` on first appearance (see [Accessibility](../accessibility.md)).
- Never rely on color alone to indicate an invalid field.
- Preserve entered values on validation failure — never clear the form.

---

## Recommended (opt-in)

- **Multi-step wizards:** keep step state in the URL (search params) rather than client-only state, so a refresh doesn't lose progress. Record the chosen approach in an ADR if it diverges from a single-page form.
- **Optimistic UI** via `useOptimistic` (React 19) for high-frequency, low-risk mutations (e.g. a like button) — not for anything that can fail validation server-side.
- **Field-level async validation** (e.g. "email already taken") debounced and called against the backend, reusing the same schema.

---

## Anti-Patterns (do not ship)

- Trusting `react-hook-form`'s client validation as the only check — always re-validate on the server.
- Building fully client-controlled forms (`useState` per field + manual `fetch`) when `react-hook-form` would do — don't hand-roll what the pairing already solves.
- Disabling the submit button as the *only* duplicate-submission guard — also guard server-side with idempotency where the action isn't naturally idempotent (see [API Resilience](../api-resilience.md)).
- Returning raw Zod error objects to the client — flatten them into a stable, presentable shape.
- Skipping `noValidate` + custom messaging and relying on inconsistent native browser validation UI across browsers.

## Quick Reference

```
✓ Shared zod schema between client and server
✓ react-hook-form + zodResolver for interactive forms
✓ Server-side re-validation on every submission path
✓ Accessible, field-associated error messages
✗ Client-only validation
✗ Hand-rolled form state when react-hook-form fits
✗ Clearing form values on validation error
```

---
*Section version: 0.1 — initial draft*
