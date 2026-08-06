# Post-Service Rating / Google Review Request (Future Feature)

## What
After a service is marked complete, automatically send the customer a WhatsApp message asking them to rate/review — pointing them to our actual Google Business Profile review page.

## Why
Client wants ratings "connected to Google" — i.e. real Google reviews, not just an internal rating stored in our own DB.

## Key constraint (important)
Google does **not** allow a third-party app to submit a review on a customer's behalf via API — a Google review must be written by the customer themselves, signed into their own Google account, on Google's UI. There is no "create review" API for us to call.

Also: Google's policy explicitly prohibits **review gating** (e.g. asking "rate us 1-5" first in our own app, then only sending happy customers onward to Google while diverting unhappy ones elsewhere). Doing this risks the Business Profile being penalized. Send everyone the same public review link, no pre-filtering.

## The two integration paths

**A. Send a direct Google review link via WhatsApp (the actual feature to build)**
- Trigger: "service completed" event → automation engine (`src/lib/automations/engine.ts`, `src/lib/automations/meta-send.ts`) sends a WhatsApp template message containing our Google review link (`https://g.page/r/<place-id>/review`).
- No Google API needed — just a link. Reuses the existing automation trigger pattern already in the codebase.
- This is the one to actually implement first.

**B. Google Business Profile API (read/reply only, optional/later)**
- Requires a verified Google Business Profile + API access.
- Lets us pull our live average rating / list of reviews into our own dashboard, and/or auto-draft replies to reviews.
- Cannot create or submit reviews — display/reply only.

## Optional complementary piece
- Separately (not instead of, and not gating access to, the Google link), we can ask a private 1-5 rating directly inside WhatsApp using interactive buttons (`sendInteractiveButtons` in `src/lib/whatsapp/meta-api.ts`), stored in our own DB as internal CSAT/feedback — independent from the public Google review.

## Build order when we get to it
1. Define the "service completed" trigger point in the automations engine.
2. Add a WhatsApp template message with the Google review link, sent via existing `meta-send.ts` path.
3. (Optional) internal 1-5 rating capture via buttons, stored separately.
4. (Optional, later) Google Business Profile API integration to surface live rating/reviews in-app.
