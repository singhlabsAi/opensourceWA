# Multi-Account Switcher — One Owner, Multiple WhatsApp Numbers (Future Feature)

## What
Let one login (one owner) manage multiple accounts — e.g. a business with 3 phone numbers, each currently needing its own separate account/login. Add an account switcher so the same person can hop between them without logging out.

## Why
Current design is a **locked one-to-one model**: `profiles.account_id` is a single column, and `accounts` has `UNIQUE(owner_user_id)` — explicitly "one account per user" (see `supabase/migrations/017_account_sharing.sql`). `whatsapp_config` also has `UNIQUE(account_id)`, so one WhatsApp number per account. Net result: a business with 3 numbers today needs 3 separate logins for the same owner. No switcher, no multi-membership exists anywhere in the codebase currently.

## Recommended approach: Option 1 — proper membership model
1. **New schema**: replace the single `profiles.account_id` column with a many-to-many `account_members (user_id, account_id, role)` table. One owner can now be a member of N accounts (one per phone number), each account still fully isolated (own contacts, broadcasts, automations, WhatsApp config) — this isolation is good, don't merge data across numbers.
2. **Active-account selector**: a session-level "which account am I viewing right now" concept (cookie/session value, not a DB column) — a dropdown in the header/sidebar like Slack's workspace switcher or Meta Business Suite's business picker. Switching should be instant, no full re-login.
3. **RLS rework required**: the `is_account_member(account_id, min_role)` helper (introduced in 017) and every policy built on it currently assume single-membership via `profiles.account_id` — all of it needs updating to check membership through the new join table instead.
4. **Invitation flow rework required**: the invitation-redemption RPC (`019_invitation_rpcs.sql`) currently *reassigns* `profiles.account_id` to the inviter's account (moves the user), rather than adding a second membership — this needs to become an "add membership" operation instead of a reassignment.

## Quick workaround (zero engineering, available today)
Owner creates 3 separate accounts using email aliases they control (e.g. Gmail `+` aliases: `owner+number1@gmail.com`, `owner+number2@gmail.com`) and just logs out/in to switch. Works now, no code changes — but no unified switcher UI, separate sessions, can't glance across all 3 accounts at once.

## Scope note
This is a genuine architecture change (schema + RLS + invitation logic), not a small patch — treat as its own project when we pick it up, not a quick add-on.
