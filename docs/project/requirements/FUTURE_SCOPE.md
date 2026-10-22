# Future Scope and Architectural Guardrails
These are directions, not authorization to implement everything now. Revisit before major design decisions.

## Planned
- Business POS, billing, sales, inventory and order APIs via a permission-aware tool gateway; optional MCP server/adapters.
- Visual workflow authoring and versioning inside customized WCRM; reusable JSON/DSL runtime if needed.
- Three-branch operations: branch comparisons, low-stock notifications, daily owner digests, supplier workflows.
- Rich customer features: bulk/wedding quotes, festival preorders, loyalty, promotions, order updates and human handoff.
- Reusable tenant-aware architecture for future unrelated client businesses, with tenant isolation and independent WhatsApp accounts.
- Multiple WhatsApp numbers, channel routing and optional multilingual support.
- Better observability: intent metrics, flow drop-offs, response quality, audit logs and cost monitoring.

## Protect future options
- Never bake branch IDs, WhatsApp phone numbers or shop names into core routing logic.
- Define tool contracts separately from LLM prompts; keep provider-specific AI logic replaceable.
- Keep tenant_id, business_id and branch_id concepts distinct; do not claim multi-tenancy is complete until isolation is tested.
- Version workflow definitions and track the active version per session.
- Avoid coupling customer conversations to internal admin dashboards or directly exposing databases to the model.
- Do not build speculative features without a stage-approved requirement and testable acceptance criteria.

## Decision review trigger
When a proposed change affects tenancy, identity, workflow state, data model, upstream WCRM integration, security or deploy strategy, read this file and write/update an ADR before coding.
