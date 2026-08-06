# Broadcasts — How It Works

## What it's for
Sending one WhatsApp message template to a list of contacts at once — a mass "campaign" send (promos, announcements, updates), as opposed to a 1:1 conversation in the inbox.

## Where it lives
- List page: `/broadcasts` → `src/app/(dashboard)/broadcasts/page.tsx`
- Create wizard: `/broadcasts/new` → `src/app/(dashboard)/broadcasts/new/page.tsx`
- Detail/report page: `/broadcasts/[id]` → `src/app/(dashboard)/broadcasts/[id]/page.tsx`
- Who can create one: requires account role **agent or above** (viewers can't) — gated via `useCan('send-messages')`.

## The 4-step creation wizard

1. **Choose Template** — only WhatsApp templates already **APPROVED by Meta** are shown (pending/rejected templates would fail at send time, so they're hidden). Shows name, category (Marketing/Utility/Authentication), body preview, language.

2. **Select Audience** — pick who receives it:
   - **All contacts**
   - **By tag** (one or more tags, OR'd together)
   - **By custom field** (field + operator `is`/`is not`/`contains` + value)
   - **CSV upload** (phone + name list; new contacts get created automatically)
   - Any of the above can also **exclude** contacts by tag.
   - A live estimated recipient count updates as you configure this.

3. **Personalize** — the template's `{{1}}`, `{{2}}`… placeholders each get mapped to either a fixed static value, a contact field (name/phone/email/company), or a custom field. If the template has an image/video/document header, you supply a media URL. A live WhatsApp-style preview shows the final message.

4. **Name & Send** — give the broadcast a name, review a summary (template, audience, estimated reach), then either **Save Draft** (creates a placeholder row, no recipients yet — you can't resume a draft back into the wizard, it's just a reminder) or **Send Now** (confirmation dialog, then it starts sending — cannot be undone).

Note: there's a "Schedule" step and `scheduled`/`scheduled_at` fields in the data model, but the actual date/time picker was removed from the UI — right now every broadcast is either a draft or sent immediately.

## What happens when you hit "Send Now"
1. The audience is resolved into an actual list of contacts.
2. A `broadcasts` row is created with status `sending`.
3. Recipient rows (`broadcast_recipients`, one per contact) are created, all starting as `pending`.
4. Recipients are sent in **batches of 10**, with a **1-second pause between batches** — this keeps you safely under Meta's per-number sending rate limits so a large broadcast doesn't get throttled or blocked by Meta.
5. Each recipient gets marked `sent` or `failed` (with an error reason) as their batch completes.
6. Once every batch is done, the broadcast's overall status becomes `sent` (as long as at least one message went through) or `failed` (if all of them failed).
7. Afterward, Meta sends back delivery/read status updates asynchronously (via webhook) which update each recipient to `delivered` → `read`, and if the contact replies, to `replied`. These roll up automatically into the broadcast's aggregate counts — you don't need to refresh anything manually.

Because this send happens from your browser tab (not a background server queue), **keep the tab open until sending finishes** for a broadcast to complete properly.

## Status meanings

**Broadcast status:**
| Status | Meaning |
|---|---|
| `draft` | Saved but not sent — no recipients created yet |
| `sending` | Actively going out right now (list page shows a live progress indicator) |
| `sent` | Finished, at least one message succeeded |
| `failed` | Finished, but every message failed (or recipient setup itself failed) |

**Per-recipient status** (always moves forward, never backward): `pending → sent → delivered → read → replied`, with `failed` as a possible dead-end from `pending` or `sent`.

## What you can see on the detail page
Open any broadcast to see:
- Stat cards: Total Recipients, Sent, Delivered, Read, Replied, Failed (with percentages)
- A funnel chart: Sent → Delivered → Read → Replied
- A full recipient table (contact, phone, status, timestamps, error message if failed) — filterable by status, exportable to CSV
- Delete (blocked while still `sending`, to avoid orphaning in-flight messages)

## Templates — the approval gate
Every broadcast requires a **Meta-approved** template. Templates are managed under Settings:
- **Create/Submit** a new template for Meta's approval (Authentication-category templates must instead be created directly in Meta Business Manager and pulled in via Sync).
- **Sync** pulls your latest templates + their real approval status straight from Meta (`APPROVED`, `PENDING`, `REJECTED`, `PAUSED`, etc.).
- **Delete** removes a template locally and, carefully, only the matching language variant on Meta's side.

Only `APPROVED` templates ever show up as selectable in the broadcast wizard — this is what actually enforces "you can only broadcast approved content."

## Rate limits & scale considerations
- You can only *start* 5 broadcast-send API calls per minute per user (protects against runaway retries/loops).
- Each send batch is capped at 10 recipients with a 1-second gap between batches.
- A single broadcast created via the public API (as opposed to the dashboard) is capped at 1,000 recipients per request — larger lists must be split into multiple requests.
- For very large contact lists, sending will simply take proportionally longer (batching + delay), not fail — just don't close the tab early.

## Known limitation
Scheduling a broadcast for a future date/time isn't currently available in the UI (the underlying data model supports it, but the picker was removed) — a broadcast is either saved as a draft or sent immediately.
