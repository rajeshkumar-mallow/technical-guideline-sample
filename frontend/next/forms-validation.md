# Forms & Validation (Next.js)

**Read this first.** This file covers Next.js-specific form submission: Server Actions and progressive enhancement. The client-rendered baseline — `react-hook-form` + `zod`, accessible error handling, shared validation schemas — lives in [`../react/forms-validation.md`](../react/forms-validation.md) and applies here too.

**Applies to:** Every form in a Next.js 15 App Router project.

---

## Mandatory

### Validate on the server regardless of client validation
Every Server Action and Route Handler re-validates input with the same schema used client-side (see [`../react/forms-validation.md`](../react/forms-validation.md)) — a request can always bypass the browser.

```ts
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

### Server Actions with `useActionState` for progressive enhancement
Prefer a plain `<form action={...}>` bound to a Server Action with `useActionState` when the form doesn't need rich client interactivity from `react-hook-form`. This works **without JavaScript** — the form still submits and re-renders with errors if the client bundle fails to load or hasn't hydrated yet.

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

When a form *does* need rich client interactivity (multi-step, dynamic fields), use `react-hook-form` per [`../react/forms-validation.md`](../react/forms-validation.md) and call the Server Action from `onSubmit` instead of relying on native form `action` binding.

---

## Recommended (opt-in)

- **Field-level async validation** (e.g. "email already taken") debounced and called via a Server Action, reusing the same schema, rather than a separate Route Handler.
- **`useOptimistic`** (see [`../react/state-management.md`](../react/state-management.md)) scoped to low-risk Server Action mutations.

---

## Anti-Patterns (do not ship)

- Skipping server-side re-validation because `react-hook-form` already validated client-side.
- Returning raw Zod error objects to the client — flatten them into a stable, presentable shape.
- Using `useActionState` + native form binding for a form that actually needs `react-hook-form`'s client interactivity (dynamic fields, inline async validation) — pick the right tool per form, not one pattern everywhere.

## Quick Reference

```
✓ useActionState + Server Actions for progressive enhancement
✓ Server-side re-validation on every submission path, regardless of client checks
✓ react-hook-form (see react/) for forms needing rich client interactivity
✗ Trusting client validation as the only check
✗ One submission pattern forced onto every form regardless of its needs
```

---
*Section version: 0.1 — initial draft*
