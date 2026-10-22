# Product Requirements — Sweet Shop WhatsApp Agent
Status: Baseline product intent; implementation not yet verified.

## Users and scope
- One sweet-shop business, three branches; customers, assigned branch managers, owner/admin and human support staff.
- WhatsApp is the primary interface; one shared number initially, with possible separate branch numbers later.
- Customers can ask about sweets, prices, ingredients, availability, locations, hours, delivery, order status, complaints, gifting and bulk orders.
- Managers can request data/actions for assigned branches only; owner can access authorized cross-branch summaries.

## Core conversation contract
1. Greeting/entry sends a welcome and three configurable quick-reply choices (e.g. Order Sweets, Track Order, Store Details).
2. A valid button reply follows a configured deterministic tree node; persist state and collected slots.
3. Free text is accepted at any time, including before button selection and during a tree.
4. For free text: check current step expectations; if it is an interruption, classify against registered flows before answering from FAQs.
5. If a confident, authorized flow matches, route to its entry/resume node and ask for missing inputs. If ambiguous, clarify rather than guessing.
6. If no flow matches, answer only from approved FAQ/knowledge sources; if unknown or sensitive, ask clarification or hand off to staff.
7. Support menu/restart/cancel/back and offer to resume a suspended workflow where appropriate.
8. Allow returning to the three-option menu after completion or on request; don't spam the menu after every response.
9. A person can repeatedly switch between buttons, natural language, and flows without losing authorized context.
10. Human handoff should suspend automation and avoid bot/staff reply collisions.

## Initial flow catalog
- `main_menu`: greeting and three buttons
- `browse_catalog`: categories, products, price/availability by branch
- `place_order`: product, quantity, branch, fulfillment, confirmation
- `track_order`: authorized lookup
- `store_details`: branches, hours, directions, contact
- `bulk_order`: event/corporate/wedding requirements and staff quote handoff
- `faq`: ingredients, freshness, storage, policies
- `complaint`: collect issue, reference, branch and escalate
- `manager_sales`: assigned-branch sales lookup
- `owner_sales`: consolidated and branch-comparison lookup

## Acceptance examples
- Customer says 'Hi' -> 3 options; customer types '5 kg kaju katli for a wedding' without clicking -> bulk-order flow.
- During catalog browsing, user asks opening hours -> answer via store-details flow and offer to resume catalog.
- Customer asks 'today's total sales' -> deny private report; offer customer assistance.
- Branch 2 manager asks for Branch 1 sales -> reject at tool authorization layer even if LLM requests it.
- Owner asks today's sales -> call live authorized API; never invent a number.
- Duplicate inbound webhook -> at most one business effect and one outbound response per event.

## Unknowns to confirm
Exact WCRM capabilities, branch names, phone number strategy, POS/inventory APIs, WhatsApp template constraints, knowledge source, supported languages, hosting details, data retention, and customer consent policy.
