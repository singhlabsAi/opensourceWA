# wacrm — Sales Feature Sheet

Self-hostable CRM for WhatsApp Business (Meta Cloud API). Use this list for pitches, landing pages, or sales calls.

| Feature | What it does | Impact for the customer |
|---|---|---|
| **Shared Team Inbox** | Multiple agents work one WhatsApp number — per-conversation assignment, status, internal notes | Stops "who's replying to this?" chaos; faster response times, no missed leads |
| **Contacts + Tags + Custom Fields** | Full contact database, CSV import, dedup | Centralizes customer data instead of scattering it across phones/spreadsheets |
| **Sales Pipelines (Kanban)** | Deals linked directly to WhatsApp conversations | Sales team tracks deal stages without leaving the chat — shorter sales cycles |
| **Broadcasts** | Meta-approved template messages, delivery/read tracking, per-recipient variable substitution | Run WhatsApp marketing campaigns at scale with proof of delivery/read — measurable ROI |
| **No-Code Automations** | Visual builder — triggers on inbound messages, new contacts, keywords, schedules; branches, waits, tags, webhooks | Customers automate follow-ups, lead routing, reminders without hiring a developer |
| **AI Reply Assistant** | Bring-your-own OpenAI/Anthropic key; one-click AI-drafted replies; optional auto-reply bot with cap + human handoff | Cuts response time and staffing cost; no per-seat AI fee — the business keeps its data |
| **AI Knowledge Base** | Upload FAQs/policies/docs; hybrid retrieval (Postgres full-text or pgvector semantic search) | Bot answers from the company's own content — accurate, on-brand support at scale |
| **Real-Time Dashboard** | Response times, daily volume, pipeline value, cross-module activity feed | Gives managers live visibility into team performance and revenue pipeline |
| **Team Accounts & Roles** | Invite by link; owner/admin/agent/viewer roles; ownership transfer; account-scoped | Lets a business scale from solo founder to full support/sales team with proper access control |
| **Account Management** | Email, password, avatar, global sign-out | Standard account hygiene/security expected by business buyers |
| **Public REST API** (`/api/v1`) | Scoped, revocable API keys | Customers can integrate the CRM into their own stack/automation tooling |
| **MCP Server** | Drive the CRM from Claude, Cursor, and other AI assistants via Model Context Protocol | Positions the product as AI-agent-ready — a strong differentiator vs. legacy CRMs |
| **Full Ownership / Self-Hosted** | Own code, own Supabase project, own domain, own data | No SaaS lock-in, no seat pricing — big cost and trust argument vs. Intercom/Zendesk-style tools |
| **Full Customisation** | Next.js + Supabase + Tailwind stack, MIT licensed | Agencies/devs can white-label and resell; businesses can add exactly the fields/workflows they need |
| **One-Click Hosting (Hostinger)** | Managed Node.js deploy, free SSL, CDN, DDoS protection, daily backups | Removes the "we don't have DevOps" objection — live in an afternoon |
| **Security Primitives** | AES-256-GCM token encryption, Postgres RLS on every table, HMAC-verified webhooks, CSP, rate limiting, CI checks on every PR | Answers enterprise/security-review objections out of the box |

## One-line pitch
"A self-hosted WhatsApp CRM your team owns outright — shared inbox, pipelines, broadcasts, no-code automations, and an AI assistant trained on your own content — deployable in an afternoon, with no per-seat fees."

## Best-fit customer segments
- Agencies/freelancers selling white-labeled CRM to SMB clients
- SMBs currently juggling WhatsApp on shared phones/spreadsheets
- Businesses wary of SaaS lock-in or per-seat AI pricing
- Teams wanting to plug WhatsApp data into AI agent workflows (Claude/Cursor via MCP)
