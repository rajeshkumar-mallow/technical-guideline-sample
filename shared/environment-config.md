# Environment & Configuration

**Read this first.** Configuration bugs that only show up in staging or production are almost always an environment-parity failure. This file defines how config is structured, named, and kept in parity across environments.

**Applies to:** every project; Next.js-specific env variable exposure risk (`NEXT_PUBLIC_*`) is covered in [react-next/security.md](../react-next/security.md).

---

## Mandatory

### Environment parity

- Define exactly four environments: `development`, `test`, `staging`, `production`. **Staging mirrors production** configuration shape (same env var names, same build mode) so config issues surface before release, not after.
- Never branch application logic on an environment name (`if (env === 'staging')`). Branch on **explicit configuration flags** instead — a staging-only feature is a flag, not an environment check, so it can be tested for real in dev too.

### File structure and naming

```
.env.example       # committed — every required var, placeholder values, comments
.env.local         # gitignored — developer's real local values
.env.test          # committed if values are non-secret test fixtures; otherwise gitignored
```

- `.env.example` is **mandatory** and kept in sync in the **same PR** that adds/removes a variable — a stale `.env.example` is documentation debt.
- Never commit a filled-in `.env.local` / `.env.production`. Add every real env file to `.gitignore` explicitly (don't rely on a wildcard alone — be explicit so it's obvious what's excluded).
- Variable names are `SCREAMING_SNAKE_CASE`, prefixed by concern where it helps clarity (`DATABASE_URL`, `STRIPE_SECRET_KEY`, `NEXT_PUBLIC_ANALYTICS_ID`).

### Secrets handling

- Secrets live in the hosting platform's secret store (Vercel/Netlify env vars, Docker secrets, cloud secrets manager) — **never in source, never in a client-shipped bundle.**
- Local development secrets (sandbox/test API keys only) go in `.env.local`. **Production credentials never touch a developer machine.**
- Scan git history before a project's first public/production deploy:
  ```bash
  npx trufflehog filesystem . --only-verified
  ```

### Typed, validated config access

Read env vars through a single validated module — not scattered `process.env.X` reads throughout the codebase:

```ts
// config/env.ts
import { z } from 'zod'

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'staging', 'production']),
  API_BASE_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().min(1),
})

export const env = envSchema.parse(process.env)
// Fails fast at boot if config is missing or malformed — not at first use, three requests in.
```

- This gives you: a compile-time type for every config value, a single source of truth, and a **fail-fast boot error** instead of a runtime `undefined` three layers deep.

---

## Recommended (opt-in)

- **Per-environment config files** (`config/staging.ts`, `config/production.ts`) merged with env vars, once a project has more than a handful of non-secret settings (feature toggles, third-party base URLs) that don't belong as raw env vars.
- **Doppler / Vault / cloud secrets manager** instead of platform-native env var UIs, once a project has more than one deploy target sharing secrets — record the choice in an ADR.

---

## Anti-Patterns (do not ship)

- `if (process.env.NODE_ENV === 'production')` scattered through business logic instead of a config flag.
- Reading `process.env.X` directly in components/services instead of through the validated `env` module.
- A `.env.example` that's missing a variable the app actually requires — new setup breaks silently.
- Committing `.env.local` or any file containing a real secret, even temporarily "to fix CI."
- Secrets baked into a client-side bundle because a var was accidentally prefixed for client exposure.

---

## Quick Reference

```
✓ 4 environments · staging mirrors production shape
✓ .env.example always current, committed · real env files gitignored explicitly
✓ Secrets in platform secret store, never in source or client bundle
✓ Config read through one validated (zod) module, fails fast at boot
✗ No Rails.env-style branching in business logic · no scattered process.env reads
✗ No committed real secrets — scan history before first prod deploy
```

---
*Section version: 0.1 — initial draft*
