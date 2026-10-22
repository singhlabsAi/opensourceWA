# Implementation Stages and Exit Gates
Do not skip gates. Each stage requires a demo, tests, updated docs and a recorded decision on outstanding risks.

## Stage 0 — Source audit and baseline
Fork/clone; inspect license, README, existing CLAUDE.md/AGENTS.md, WCRM architecture, webhook, AI, workflow, API, MCP, database and Docker. Inventory supported vs custom capabilities. Establish baseline tests and merge documentation without overwriting useful upstream instructions.
Exit: source-verified integration map, reproducible local startup, known gaps.

## Stage 1 — Git and local environment
Configure origin/upstream, main/develop/feature PR policy, local env, sample secrets, lint/test commands.
Exit: repeatable development environment and CI checks.

## Stage 2 — WCRM branding and three-branch configuration
Minimal UI branding, tenant/business and branch configuration, contact mapping, test WhatsApp number.
Exit: inbound/outbound test messages, branch data model, no secret exposure.

## Stage 3 — Deterministic WhatsApp workflows
Greeting, three-button menu, catalog, branch details, order inquiry; persistent flow state, buttons, back/menu/cancel.
Exit: end-to-end scripted tests and duplicate event protection.

## Stage 4 — AI intent router and FAQ
Interruptible free text, flow matching, confidence/clarification, FAQ retrieval, suspend/resume, human handoff.
Exit: scenario tests including ambiguous, out-of-flow and unknown questions.

## Stage 5 — RBAC and owner/manager reports
Provision verified principals, branch scopes, mock sales tool, deny-by-default authorization, audit.
Exit: unauthorized role/branch tests pass; authorized mock data demo.

## Stage 6 — Business integrations and MCP
Connect real POS/order/inventory APIs, enforce scoped tools, read vs write permissions, confirmation and idempotency.
Exit: integration tests, degraded-service behavior, source-of-truth validation.

## Stage 7 — VPS production and delivery automation
Docker deployment, TLS, secrets, persistent volumes, health checks, backups, CI/CD, staging, rollback.
Exit: production smoke test and successful restore drill.

## Stage 8 — Upstream sync and operations
Scheduled eligibility check for 15-day sync, fetch upstream, open PR, test and review conflicts; observability and incident runbook.
Exit: simulated upstream change PR and no automatic unreviewed production merge.

## Stage 9 — Future product expansion
Visual editor extensions, more branches/numbers, campaigns, analytics, tenant isolation, broader clients. Prioritize via validated customer demand.
