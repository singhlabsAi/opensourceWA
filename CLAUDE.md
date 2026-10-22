@AGENTS.md

# WCRM AI Project — Claude Code Entry Point

## Mission
Self-hosted, customized WCRM WhatsApp AI operations platform. First deployment: a three-branch sweet shop. Combine deterministic button/tree workflows, AI intent-to-flow routing, FAQ fallback, human handoff and role/branch-authorized business tools, keeping a path to multi-tenant.

## Read selectively (minimize tokens)
1. Always read this file and `docs/project/delivery/CURRENT_STAGE.md`.
2. Before changing behavior: `docs/project/requirements/PRODUCT.md`, `docs/project/architecture/CONVERSATION_ENGINE.md`.
3. Permissions, API/MCP tools, data access: `docs/project/architecture/SECURITY.md`.
4. Hosting, CI/CD, upstream sync: `docs/project/delivery/OPERATIONS.md` and `knowledge/`.
5. Design changes / major refactors: `docs/project/requirements/FUTURE_SCOPE.md`, `docs/project/decisions/`.
6. Read only relevant source files and adjacent tests.

## Non-negotiable invariants
- WCRM is the transport/inbox/CRM foundation; keep custom orchestration modular. Prefer extensions/adapters over editing upstream core.
- Button payloads deterministically advance an authorized workflow. Free text: active-step interpretation → intent-to-flow routing → approved FAQ → human handoff.
- One conversation router for all incoming events; idempotent webhook handling, no duplicate replies.
- Authorization enforced server-side per operation (role + branch scope); a customer message can never grant elevated access.
- AI never fabricates sales, stock, prices, order status or policy.
- State-changing actions need explicit confirmation, authorization and an audit record.
- Never commit secrets, tokens, customer PII or production data.

## Workflow rules
- Follow the phase/stage gates in `docs/project/delivery/STAGES.md`; never skip or declare complete without evidence.
- Before implementing: identify stage, affected modules, conflicts with invariants/ADRs; state acceptance criteria. Challenge conflicting requests.
- No deploys, pushes, destructive commands or production changes without showing a plan and getting approval. Never auto-merge upstream into production.
- After implementing: run `npm run lint`, `npm run typecheck`, `npm test` (and `npm run build` when relevant); update `CURRENT_STAGE.md`.

## Documentation map
Ask the user before writing or editing any doc; update an existing file rather than creating a duplicate.
- `docs/project/` — this customization's requirements, architecture, delivery stages and ADRs.
- `docs/` (top level) — reference for features that already ship (Automations, Broadcasts, mcp, MetaPricing, public-api, sales-features).
- `FutureFeature/` — unbuilt ideas with rationale/scope.
- `knowledge/` — deployment, hosting and local-setup notes.
