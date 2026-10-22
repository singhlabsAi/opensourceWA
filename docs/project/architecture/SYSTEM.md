# System Architecture (Target, not yet source-verified)

WhatsApp user -> Meta Cloud API -> WCRM inbound event -> verified event adapter -> conversation router -> workflow runtime OR intent matcher -> approved FAQ / business tool gateway / human handoff -> WCRM outbound API -> WhatsApp.

## Boundaries
- WCRM: transport, shared inbox, contact management, existing automation UI and operational CRM features as verified in source.
- Event adapter: signature verification, normalization, deduplication, retry safety and outbound idempotency.
- Conversation router: identity lookup, current session, event type, commands, routing precedence and handoff.
- Workflow runtime: versioned definitions, state, slots, button IDs, transitions, suspended flows, timeout/expiry.
- AI layer: bounded intent classification and grounded FAQ answers; structured outputs validated by code.
- Tool gateway: independently authorizes every call, applies tenant/branch scoping, logs access and enforces confirmations.
- Data: separate operational business data from WCRM's own contact/inbox data; define migrations only after source review.

## Integration strategy
Inspect WCRM's existing webhook, AI, workflow, REST, MCP, auth, and database implementations before selecting integration points. Prefer adapters and extension points to invasive forks. Do not assume feature support based on README alone.

## Operational qualities
Idempotency, queue/retry strategy, rate limits, error fallbacks, audit trails, structured logs, metrics, secrets management, backup/restore and safe rollout. Handle concurrent messages in a single conversation deterministically.
