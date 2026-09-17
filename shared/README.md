# Shared Guidelines

**Truly language-agnostic** rules — true whether the backend is Rails, Python, PHP, or Node, and whatever the frontend framework. Nothing here assumes JavaScript. JS/TS-specific rules that apply across any JS runtime live in [`../js/`](../js/README.md); browser-specific rules live in [`../frontend/`](../frontend/README.md). Every project reads this folder first.

| File | Covers |
|------|--------|
| [Git & PR Workflow](git-pr-workflow.md) | Branching strategy, commit conventions, PR review checklist |
| [Testing Philosophy](testing-philosophy.md) | Test pyramid, coverage expectations, unit vs. functional vs. e2e definitions |
| [Setup Checklist](setup-checklist.md) | Prerequisites, install steps, first-PR checklist |
| [Approved Libraries](approved-libraries.md) | Pre-approved packages, criteria for adding a new dependency |
| [Environment & Configuration](environment-config.md) | `.env` structure, dev/staging/prod parity, secrets handling |
| [Deployment](deployment.md) | CI/CD pipeline, hosting options, build/deploy gates |
| [Project Architecture](project-architecture.md) | Folder structure conventions; monolith vs. micro-frontend decision tree |
| [Documentation Standards](documentation-standards.md) | JSDoc/TSDoc, README, Storybook doc expectations |
| [Versioning & Release Process](versioning-release-process.md) | Semver, changelogs, version-currency review trigger |
| [Feature Flags](feature-flags.md) | Rollout strategy, flag hygiene |
| [Privacy & Data Compliance](privacy-compliance.md) | Cookie consent, client-side PII handling |

Back to [root guideline](../README.md).
