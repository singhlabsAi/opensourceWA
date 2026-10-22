# Current Stage — Living Project Checkpoint
Updated: 2026-10-22
Current: Phase 1 — GitHub setup and repository audit (in review)

## Verified (2026-10-22)
- Fork `singhlabsAi/opensourceWA` → upstream `ArnasDon/wacrm` (MIT). Remotes: `origin` = fork, `upstream` = fetch-only (push disabled).
- Fork was 24 ahead / 278 behind upstream. A trial merge shows conflicts in `.dockerignore`, `.gitignore`, `Dockerfile`, `docker-compose.yml`, `messages/en.json`, `src/components/automations/automation-builder.tsx`.
- Scaffold docs merged into `docs/project/`; root `CLAUDE.md`/`AGENTS.md` merged (the fork's Next.js rule and doc map are kept). CODEOWNERS now `@singhlabsAi`.
- Existing CI `ci.yml` (lint/typecheck/test/build). `upstream-watch.yml` pushes straight to `main` — replace in Phase 4.

## Decisions (see ADR-002)
- Database: hosted Supabase free tier. VPS access via SSH; credentials only in GitHub Actions secrets. Domain and WhatsApp test number come later.

## Next actions
1. Review/merge the scaffold PR.
2. Upstream sync PR (`sync/upstream-20261022`), resolving the 6 conflicts while keeping fork customizations.
3. Create `develop`, delete stale `setup_info`, enable branch protection (after upstream-watch is fixed).
4. Phase 2: local run against the Supabase project (needs `.env.local` from the user).

## Blockers
Domain, WhatsApp test number, POS/inventory APIs: pending from the user.
