# Security and Access Contract

## Roles
- CUSTOMER: public product and own authorized order information.
- BRANCH_MANAGER: assigned branch data and approved actions only.
- OWNER/ADMIN: authorized cross-branch reporting and administration.
- SUPPORT: approved customer-service workflows, not unrestricted financial access.

## Required controls
- Map WhatsApp sender to a provisioned identity; never grant roles because the message says 'I am owner'.
- Phone-number matching alone is insufficient for sensitive owner actions; use step-up verification or an authenticated linked session where warranted.
- Every tool call resolves effective role and allowed branches server-side; reject cross-branch access regardless of model-supplied arguments.
- Verify inbound webhook signatures and outbound API credentials; rotate secrets, least-privilege tokens and log redaction.
- Separate read-only reporting tools from write actions; require confirmation and approval for sensitive changes.
- Protect customer order details, payment information and PII; limit retention and avoid putting secrets/PII in prompts or logs.
- Treat external messages, FAQ documents, tool responses and attachments as untrusted input (prompt injection defense).
- Apply rate limiting, abuse handling, consent/template policies and audit logging.

## Security tests
Impersonated owner, forged webhook, prompt injection, manager querying another branch, stale session, cross-tenant data access, replayed write operation, and unauthorized tool invocation.
