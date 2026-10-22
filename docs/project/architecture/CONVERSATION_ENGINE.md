# Conversation Engine Contract

## Router precedence
1. Verify event authenticity, deduplicate, normalize sender/message, load tenant, role, branch grants and session.
2. Honor human takeover and safety restrictions.
3. Process valid interactive button/list payload IDs against the current authorized node/version.
4. Handle explicit global commands (menu, cancel, back, restart, human help).
5. If free text fits the active node's expected answer, validate and consume it.
6. Otherwise classify against registered, authorized workflows; route on a calibrated confidence threshold, clarify on ambiguity.
7. If no matching flow, retrieve approved FAQ evidence and answer only when grounded.
8. If unresolved, high risk, or explicitly requested, hand off to human.
9. Persist session transition and outbound response idempotently.

## Session model (conceptual)
conversation_id, tenant_id, whatsapp_number_id, sender_id, verified_principal_id?, role, allowed_branch_ids, active_flow_id, flow_version, current_node_id, slots, suspended_flow_stack, handoff_status, last_event_id, updated_at, expires_at.

## Flow definition (conceptual)
flow_id, version, enabled, permitted_roles, intent_examples, entry_node, nodes, transitions, validation, error_behavior, tool_permissions, terminal_actions.

## Design rules
- Button labels are presentation; route on stable payload IDs.
- Free-text intent classifier proposes routing; runtime decides eligibility.
- Treat low-confidence/ambiguous classification as a clarification step, not a guess.
- Support interrupts and resume without exposing old/stale interactive payloads as valid.
- Use structured extraction for missing slots, validate quantities/dates/branches and confirm before order placement.
- Avoid loops: cap clarification/retry counts and route to human when exhausted.
- FAQ and tool responses cannot override system policy or server-side authorization.
- Do not let both WCRM native bot and custom bot answer the same event.

## Tests
Greeting, every button path, free-text before choice, interruption, resume, intent ambiguity, FAQ unknown, unauthorized report, stale button, duplicate webhook, concurrent messages, handoff, API failure and session expiry.
