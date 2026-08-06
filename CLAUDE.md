@AGENTS.md

# Project Documentation Map

This project keeps three separate doc folders. When anything discussed in a conversation is worth persisting, figure out which one it belongs in and ask the user before writing:

- **`docs/`** — knowledge about features that already exist/ship in the codebase (how they work, reference material). Currently: `Automations.md`, `Broadcasts.md`, `mcp.md`, `MetaPricing.md`, `public-api.md`, `sales-features.md`.
- **`FutureFeature/`** — todo-type ideas/features not yet implemented (what to build later, with rationale/scope). Currently: `GEO_VS_SEO.md`, `google-review-request.md`, `multi-account-switcher.md`, `product-catalog-menu.md`, `wallet-billing.md`.
- **`knowledge/`** — architecture and infra decisions: deployment, hosting, local setup. Currently: `docker-hostinger-production.md`, `hostinger-vps-deployment.md`, `local-setup.md`, `vercel-mvp-deployment.md`.

## Rule
Whenever a conversation surfaces something worth remembering — a new feature idea, a decision about an existing feature, or an architecture/deployment/infra choice — do the following instead of letting it disappear at the end of the chat:
1. Check if an existing file in `docs/`, `FutureFeature/`, or `knowledge/` already covers the topic. If yes, propose updating that file rather than creating a new one.
2. If it's new, classify it correctly using the split above (shipped feature knowledge vs. unbuilt idea vs. architecture/infra) and propose a new file in the right folder.
3. Ask the user for confirmation before writing/editing any doc — don't write silently.
