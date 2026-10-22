# Git, Deployment and Upstream Operations

## Git
`origin` = owned fork; `upstream` = original WCRM. Use short-lived feature branches, PR checks, reviewed merges into develop/main, and branch protection. Avoid force-pushing shared branches. Preserve upstream attribution and license.

## Deployment target
User-owned VPS. Target containerized services behind HTTPS reverse proxy, non-root processes where possible, least-privilege secrets, persistent database/storage, health checks, metrics, offsite backups and restore drills. Finalize stack only after auditing WCRM's actual Docker and database requirements.

## CI/CD
PR: lint, typecheck, unit/integration tests, dependency/secrets checks, build. Main merge: build immutable image, publish registry artifact, deploy via SSH/runner with protected environment and secrets, health-check, and rollback on failure. Use staging for migrations and risky updates.

## Upstream synchronization
Requirement: check eligibility daily; if >=15 days since last successful sync review, fetch upstream and prepare a dedicated sync PR. Do not use `*/15` as a true every-15-days guarantee. Keep a durable last-sync record and handle missed runs. CI tests, inspect upstream diffs and migration notes, resolve conflicts, manual approval; never deploy upstream changes straight to production.

## Release gate
Check data migrations, backups, rollback, changed environment variables, WhatsApp webhook configuration, AI costs, authorization regressions and conversation scenario tests before deployment.
