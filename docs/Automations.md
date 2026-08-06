# Automations — How It Works

## What it's for
"When X happens, automatically do Y" — single-fire reaction rules. Not a multi-turn conversation builder (that's the separate "Flows" tab, see below) — an Automation reacts once to a trigger event and runs a sequence of actions.

## Where it lives
- List: `/automations` → `src/app/(dashboard)/automations/page.tsx`
- Create: `/automations/new` (optionally pre-filled from a starter template)
- Edit: `/automations/[id]/edit`
- Run history: `/automations/[id]/logs`
- Requires account role **agent or above** to create/activate (same gate as Broadcasts, since automations can send WhatsApp messages).

## Building one

**1. Pick a trigger** — the event that starts the automation:
| Trigger | Fires when |
|---|---|
| `new_message_received` | Any inbound WhatsApp message |
| `first_inbound_message` | The very first message ever from this contact |
| `keyword_match` | Inbound message text matches configured keyword(s) — contains or exact, case-sensitive optional |
| `interactive_reply` | Customer taps a specific button/list option (by reply id) |
| `new_contact_created` | A new contact record is created |
| `tag_added` | A specific tag gets added to a contact (manually or by another automation) |
| `conversation_assigned` | *(available in the builder, but currently never actually fires — see Known Limitations)* |
| `time_based` | *(same — configurable in the builder, but not wired to run yet)* |

**2. Add steps** — the sequence of actions to run, in order, editable/reorderable:
| Step | What it does |
|---|---|
| `send_message` | Sends a free-text reply (supports `{{message.text}}` / `{{vars.x}}` placeholders) |
| `send_template` | Sends an approved WhatsApp template with variables |
| `send_buttons` / `send_list` | Sends an interactive button or list message |
| `add_tag` / `remove_tag` | Tags/untags the contact — adding a tag can itself trigger other `tag_added` automations (chained, depth-capped to prevent loops) |
| `assign_conversation` | Assigns the conversation to a specific agent or a round-robin pick |
| `update_contact_field` | Updates name/email/company or a custom field |
| `create_deal` | Creates a pipeline deal for the contact |
| `wait` | Pauses the automation for N minutes/hours/days, then continues later |
| `condition` | Branches into "Yes"/"No" paths based on tag presence, a contact field, message content, or time of day — each branch has its own steps |
| `send_webhook` | POSTs to your own URL with a custom payload (blocked from targeting internal/private addresses for safety) |
| `close_conversation` | Marks the conversation closed |

**3. Save & activate** — turning it on runs validation first (e.g. you can't activate a `send_message` step with empty text, or a `wait` with zero duration).

Starter templates exist for common cases and pre-fill the trigger + steps for you when you have fewer than 3 automations set up.

## What happens when it fires (step by step)
1. A customer sends a message (or another qualifying event happens).
2. The WhatsApp webhook (or the relevant internal event) looks up all **active** automations for your account matching that trigger type.
3. For triggers with extra config (`keyword_match`, `interactive_reply`, `tag_added`), the engine double-checks the actual content/id against what you configured — only real matches run.
4. A log entry is created, and the automation's steps run in order: sends go out via the same Meta WhatsApp API broadcasts uses, tag/field changes are written immediately, conditions branch into the matching sub-path.
5. If a `wait` step is hit, the automation **pauses** — it's parked and picked back up later by a scheduled background check, then resumes exactly where it left off (no need to keep any tab open, unlike Broadcasts).
6. When finished, the run is marked `success`, `partial` (e.g. still waiting), or `failed`, and the automation's execution counter/last-run time update.
7. Messages an automation sends appear in the normal inbox thread, just tagged as bot-sent rather than agent-sent, so conversation history stays unified.

## Viewing what happened
Open **View Logs** on any automation to see, per run: which contact triggered it, which trigger fired, a status badge (success/partial/failed), and a step-by-step trace — e.g. "sent via Meta (message id ...)", "waiting 10 minutes", "branch=yes", or the specific error if a step failed. This is a faithful record of exactly what the engine did, useful for debugging why something didn't send as expected.

## Automations vs. Flows — don't confuse them
- **Automations** = single-fire reaction rules, no persistent per-contact conversation state.
- **Flows** (separate beta tab) = stateful, multi-turn guided conversations/menus.
- **Priority rule**: if a contact is actively navigating a Flow menu, content-based automation triggers (`new_message_received`, `keyword_match`) are **suppressed** for that message — the assumption is they're navigating your bot menu, not sending a fresh trigger phrase. Relationship-based triggers (`new_contact_created`, `first_inbound_message`) still fire regardless.
- They share the same underlying interactive-message sending code, so buttons/lists behave identically whether sent from an Automation or a Flow.
- Rule of thumb: simple auto-reply / tagging / routing → Automations. A branching multi-step guided conversation → Flows.

## Known limitations (worth knowing before relying on them)
- **`conversation_assigned` and `time_based` triggers can be configured in the builder and saved/activated, but nothing in the system currently ever fires them.** An automation built on either of these will simply never run. Treat these two as "not yet implemented" rather than usable options for now.
- Round-robin agent assignment doesn't yet do true round-robin — it currently just picks an arbitrary account member.
- `send_webhook` has a 10-second timeout and blocks private/internal URLs as a safety measure — it can only reach public endpoints.
