# Forms & Validation (Frontend)

**Read this first.** Forms are where most accessibility and data-integrity bugs live — silent validation failures, errors announced to nobody, and client/server rule drift. This file sets the framework-agnostic baseline; React/Next-specific form patterns (React Hook Form, Server Actions) are in [`react-next/forms-validation.md`](../react-next/forms-validation.md).

**Applies to:** any HTML form, in any stack.

---

## Mandatory

### Native semantics first
- Use real `<form>`, `<label>`, `<input>`, `<select>`, `<textarea>` — never `<div>`s wired up with JS to fake form controls.
- Every input has a **programmatically associated label** (`<label for>` / wrapping `<label>`). Placeholder text is not a label.
- Use the correct `type` and `inputmode` (`email`, `tel`, `number`, `url`) so browsers and assistive tech get native validation and correct mobile keyboards for free.
- Mark required fields with the `required` attribute, not just visual styling — and don't rely on it alone (see below).

### Accessible error messaging
Errors must be perceivable by screen reader users at the point of the mistake, not just visually near it.

```html
<label for="email">Email</label>
<input
  id="email"
  name="email"
  type="email"
  aria-invalid="true"
  aria-describedby="email-error"
/>
<p id="email-error" role="alert">Enter a valid email address.</p>
```

- `aria-describedby` links the input to its error message.
- `aria-invalid="true"` is set only while the field is actually invalid — remove it once corrected.
- `role="alert"` (or an `aria-live="polite"` region for a summary) ensures the error is announced without moving focus unexpectedly.
- On submit failure, move focus to the **first invalid field** or to an error summary — never leave focus stranded on a disabled submit button.

### Validation layering
1. **Native HTML constraints** (`required`, `type`, `pattern`, `minlength`) — the cheapest first line, works even if JS fails to load.
2. **Client-side schema validation** for real UX (Zod, Yup, or equivalent) — validate on blur/submit, not on every keystroke (keystroke-level validation for required-ness reads as hostile UI).
3. **Server-side validation is mandatory regardless of client validation.** Client validation is a UX convenience, never a security or data-integrity boundary — assume it can be bypassed.

```ts
import { z } from "zod";

const SignupSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

// Same schema, reused server-side — one source of truth for the rule.
```

### Submission state
- Disable the submit control (or show a spinner) while a request is in flight; prevent double-submit.
- On network/server error, show a recoverable message — never a dead end (ties to [`api-resilience.md`](api-resilience.md)).
- Preserve entered values on validation failure — never clear the form.

---

## Recommended (opt-in)

- **Debounced async validation** (e.g. "username taken" checks) — 300–500ms debounce, cancel in-flight requests on new input.
- **Multi-step / wizard forms** — persist progress (session storage or backend draft) so a refresh doesn't lose data; record the chosen persistence strategy via ADR if it touches sensitive data (see [`privacy-compliance.md`](../shared/privacy-compliance.md)).
- **Optimistic UI** for low-risk mutations only — never for anything money- or identity-related.

---

## Anti-Patterns (do not ship)

- Divs styled as inputs with `onClick`/`onKeyDown` reimplementing native form behavior.
- Validating only on the client and trusting the payload server-side.
- Error text with no programmatic association to its field (a red `<span>` near the input isn't enough).
- Clearing the whole form on a single field's validation error.
- Validating every keystroke for `required` fields before the user has had a chance to finish typing.

---

## Quick Reference

```
✓ Real <label>/<input>, aria-describedby + role="alert" for errors
✓ Native constraints → client schema validation → mandatory server validation
✓ Focus moves to first invalid field on failed submit
✓ Disable submit / prevent double-submit while in flight
✗ Fake form controls built from divs
✗ Client-only validation trusted as the data-integrity boundary
✗ Clearing form state on validation error
```

---
*Section version: 0.1 — initial draft*
