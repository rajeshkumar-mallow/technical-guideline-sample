# Deployment

**Read this first.** Deployment is where every other gate (lint, types, tests, security) either gets enforced or quietly skipped. This file defines the mandatory pipeline stages and treats specific hosting choices as opt-in packs, the same way the platform choice is a project decision, not a guideline mandate.

**Applies to:** every project. See [Environment & Configuration](environment-config.md) for how secrets reach these environments.

---

## Mandatory

### Pipeline stages (every project, regardless of host)

Every merge to the default branch — and every deploy — runs through these gates, **in this order**, and a failure at any stage blocks the deploy:

| Stage | Command / check | Blocks deploy on failure? |
|---|---|---|
| Install | `pnpm install --frozen-lockfile` | Yes |
| Lint | `pnpm lint` | Yes |
| Type check | `pnpm typecheck` | Yes |
| Unit/component tests | `pnpm test` | Yes |
| E2E (at least smoke) | `pnpm test:e2e` | Yes for production deploys |
| Build | `pnpm build` | Yes |
| Security audit | `pnpm audit --audit-level=high` | Yes |
| Bundle size check | project's size-limit script | Warn or block per project addendum |

- Local and CI gates must be **identical commands** — if `pnpm lint` passes locally but CI runs something different, that's a bug in the pipeline, not an exception to fix around.
- **Preview deployments** for every PR (Vercel preview URLs, or equivalent) are mandatory once a project ships UI — reviewers verify the actual rendered change, not just the diff.
- **Production deploys require the full gate**, including e2e. Staging/preview deploys may skip e2e for speed but must run it before promotion to production.
- **Rollback path** must exist and be documented — either instant rollback to the previous deployment (typical on Vercel/Netlify) or a documented redeploy-previous-tag procedure for container-based hosts.

### Environment promotion

```
feature branch → preview deploy (every PR)
        │
        ▼
     staging  (auto-deploy on merge to main; mirrors prod config)
        │
        ▼
   production (manual promote, or auto after staging smoke tests pass)
```

- Never deploy directly to production from a feature branch, "just this once."
- Database/schema migrations (if the project has a backend) run as a **separate, reviewed step** before the app deploy that depends on them — never bundled invisibly into the app build.

---

## Recommended (opt-in) — hosting packs

Pick one; record the choice and reasoning in the project addendum. All application-facing pipeline requirements above still apply regardless of host.

| Pack | Fits when |
|---|---|
| **Vercel** | Next.js projects wanting zero-config previews, edge functions, and image optimization out of the box. Default recommendation for greenfield Next projects. |
| **Docker + generic host (Fly.io/Render/ECS/K8s)** | Project needs infra control, isn't Next-only, or has non-JS services alongside the frontend. |
| **Static export + CDN** (S3+CloudFront, Netlify, Cloudflare Pages) | Fully static/SSG site with no server runtime needs. |

- **CI/CD provider:** GitHub Actions is the default; document any deviation (GitLab CI, CircleCI) in the project addendum.
- **Infrastructure-as-code** for anything beyond a single-service deploy (Terraform, Pulumi) — adopt once manual console clicking becomes a repeated source of drift.

---

## Anti-Patterns (do not ship)

- Skipping CI on a "trivial" change via a direct push to the deploy branch.
- Different lint/test commands locally vs. in CI.
- Deploying to production without a rollback path tested at least once.
- Running database migrations as an implicit side effect of app boot in production.
- No preview deployment for UI changes — reviewing a screenshot pasted in the PR instead of the real thing.

---

## Quick Reference

```
✓ install → lint → typecheck → test → e2e → build → audit, every merge, same commands local+CI
✓ Preview deploy per PR · staging mirrors prod · production requires full gate incl. e2e
✓ Documented, tested rollback path · migrations run as a separate reviewed step
✓ Hosting choice recorded in project addendum, pipeline requirements apply regardless of host
✗ No direct-to-prod pushes · no skipped CI "just this once" · no undocumented rollback
```

---
*Section version: 0.1 — initial draft*
