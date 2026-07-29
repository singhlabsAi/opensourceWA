# Vercel MVP Deployment Guide — opensourceWA (wacrm)

Step-by-step guide to deploy this WhatsApp CRM on **Vercel** for an MVP — no VPS, no Docker, no Nginx.

**Estimated time:** 30–45 minutes (first time)

**Stack:**

- App: Next.js 16 on Vercel (serverless)
- Database: Supabase (hosted — not on Vercel)
- WhatsApp: Meta Cloud API webhooks
- Domain: Vercel's free `*.vercel.app` subdomain is fine for MVP (custom domain optional)

---

## What you get on MVP

| Works out of the box | Skip for MVP (add later) |
|---|---|
| Sign up / login | Custom domain |
| Shared inbox, contacts, pipelines | Automation **Wait** steps (needs cron) |
| WhatsApp send & receive | Flow timeout sweep cron |
| Broadcasts, automations (no Wait steps) | AI assistant (BYOK — optional per account) |
| HTTPS + auto-deploy on git push | MCP server (runs against your deployed URL) |

---

## Pre-flight checklist

Gather these before you start:

| Item | Where to get it |
|---|---|
| GitHub account | [github.com](https://github.com) |
| Vercel account | [vercel.com](https://vercel.com) (sign in with GitHub) |
| Supabase project URL | Supabase → Settings → API |
| Supabase anon key | Supabase → Settings → API |
| Supabase service_role key | Supabase → Settings → API |
| Meta App Secret | Meta Developers → App Settings → Basic |
| WhatsApp Phone Number ID | Meta → WhatsApp → API Setup |
| WhatsApp Business Account ID | Meta → WhatsApp → API Setup |
| WhatsApp Access Token | Meta System User token (recommended) |

Generate secrets on your laptop:

```bash
# ENCRYPTION_KEY (64 hex chars) — save permanently
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Webhook verify token (any random string you choose)
openssl rand -hex 16
```

> **Important:** Save `ENCRYPTION_KEY` permanently. Changing it breaks stored WhatsApp tokens.

---

## Part 1 — Supabase setup

### 1.1 Create Supabase project

1. Go to [supabase.com](https://supabase.com) → **New project**
2. Pick a region close to your users (e.g. Mumbai / Singapore for India)
3. Save the database password

### 1.2 Run database migrations

All migrations are in `supabase/migrations/`. Apply **in numeric order** (`001` through `036`).

**Option A — SQL Editor (easiest for MVP):**

1. Supabase dashboard → **SQL Editor**
2. Open each file locally, paste, run
3. Repeat for all 36 files

**Option B — Supabase CLI:**

```bash
npm install -g supabase
supabase login
supabase link --project-ref housmckjeimhzkbvfobg
supabase db push
```

### 1.3 Verify tables

In **Table Editor**, confirm these exist:

- `profiles`, `contacts`, `conversations`, `messages`
- `pipelines`, `broadcasts`, `automations`, `whatsapp_config`

### 1.4 Copy API keys

From **Project Settings → API**, copy:

- Project URL → `NEXT_PUBLIC_SUPABASE_URL`
- anon / public key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- service_role key → `SUPABASE_SERVICE_ROLE_KEY`

Keep the service_role key secret — never expose it in client code.

---

## Part 2 — Push code to GitHub

Vercel deploys from Git. If your repo is not on GitHub yet:

1. Fork or push this repo to your GitHub account
2. Ensure the default branch is `main` (or note which branch you will deploy)

You do **not** need a `Dockerfile` or `docker-compose.yml` for Vercel — Vercel builds Next.js natively.

---

## Part 3 — Create the Vercel project

### 3.1 Import the repo

1. Go to [vercel.com/new](https://vercel.com/new)
2. **Import** your GitHub repository
3. Vercel auto-detects **Next.js** — leave framework preset as-is

### 3.2 Build settings (defaults are correct)

| Setting | Value |
|---|---|
| Framework Preset | Next.js |
| Root Directory | `./` (repo root) |
| Build Command | `npm run build` |
| Output Directory | (auto — leave blank) |
| Install Command | `npm install` |

Node.js version: Vercel uses Node 20+ by default, which matches this project's `engines` requirement.

### 3.3 Add environment variables

Before clicking **Deploy**, expand **Environment Variables** and add:

#### Required

| Name | Value |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | `https://your-project.supabase.co` |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | your anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | your service_role key |
| `ENCRYPTION_KEY` | your 64-char hex key |
| `META_APP_SECRET` | Meta App Secret |

#### Recommended

| Name | Value |
|---|---|
| `NEXT_PUBLIC_SITE_URL` | `https://your-project.vercel.app` (update after first deploy — see 3.5) |
| `NEXT_PUBLIC_APP_LOCALE` | `en` |
| `META_APP_ID` | Meta App ID (needed for image-header templates) |

> Set each variable for **Production** (and Preview/Development if you want preview deploys to work fully).

### 3.4 Deploy

Click **Deploy**. First build takes 2–5 minutes.

When it finishes, Vercel gives you a URL like:

```
https://your-project.vercel.app
```

Open it — you should see the login/signup page.

### 3.5 Set the canonical URL

After you know your Vercel URL:

1. Vercel → **Project → Settings → Environment Variables**
2. Set or update `NEXT_PUBLIC_SITE_URL` to `https://your-project.vercel.app` (no trailing slash)
3. **Redeploy** (Deployments → ⋯ → Redeploy) so the new value is baked in

---

## Part 4 — Supabase auth URLs

Supabase → **Authentication → URL Configuration**:

| Field | Value |
|---|---|
| Site URL | `https://your-project.vercel.app` |
| Redirect URLs | `https://your-project.vercel.app/**` |

If you add a custom domain later, update both values to match.

**Optional for MVP:** Under **Authentication → Providers → Email**, turn off **Confirm email** so sign-up is instant during testing.

---

## Part 5 — Meta WhatsApp webhook

### 5.1 Meta app setup (if not done yet)

1. [Meta for Developers](https://developers.facebook.com/) → **My Apps → Create App**
2. Choose **Business** type
3. Add the **WhatsApp** product
4. On **WhatsApp → API Setup**, note Phone number ID, WABA ID, and access token

For a token that does not expire in 24 hours, create a **System User** token with:

- `whatsapp_business_management`
- `whatsapp_business_messaging`

### 5.2 Configure webhook

Meta for Developers → **WhatsApp → Configuration**:

| Field | Value |
|---|---|
| Callback URL | `https://your-project.vercel.app/api/whatsapp/webhook` |
| Verify token | The random string you chose in pre-flight |

Click **Verify and save**. If verification fails, see [Troubleshooting](#troubleshooting) below.

Subscribe to webhook fields:

- `messages`
- `message_template_status_update`
- `message_template_quality_update`
- `message_template_components_update`

Ensure `META_APP_SECRET` in Vercel matches Meta → **App Settings → Basic → App Secret**.

---

## Part 6 — Connect WhatsApp in the app

1. Open `https://your-project.vercel.app`
2. **Sign up** (first user becomes the account owner)
3. Go to **Settings → WhatsApp**
4. Enter:
   - Phone number ID
   - WABA ID
   - Access token
   - Verify token (same string as Meta webhook)
5. **Save**

Tokens are encrypted with `ENCRYPTION_KEY` before storage in Supabase.

---

## Part 7 — Smoke test

- [ ] Site loads at your Vercel URL
- [ ] Can sign up / log in
- [ ] Dashboard loads without 500 errors
- [ ] WhatsApp settings save successfully
- [ ] Meta webhook shows green checkmark (verified)
- [ ] Send a test message from Meta → appears in **Inbox**
- [ ] Reply from the app → recipient gets the WhatsApp message

---

## Part 8 — Deploying updates

Every push to the connected branch triggers a new Vercel deployment automatically.

Manual redeploy: Vercel → **Deployments** → ⋯ → **Redeploy**

**Before deploying code that adds migrations:**

1. Apply new SQL files in `supabase/migrations/` (in numeric order) via Supabase SQL Editor
2. Then push to GitHub (or redeploy)

---

## Optional — custom domain

For MVP you can stay on `*.vercel.app`. When ready:

1. Vercel → **Project → Settings → Domains** → add `crm.yourdomain.com`
2. Add the DNS records Vercel shows (usually a CNAME)
3. Update `NEXT_PUBLIC_SITE_URL` to `https://crm.yourdomain.com`
4. Update Supabase auth URLs (Site URL + Redirect URLs)
5. Update Meta webhook Callback URL
6. Redeploy

SSL is automatic on Vercel — no Certbot needed.

---

## Optional — automation cron (Wait steps & flows)

Skip this for MVP unless you use **Wait** steps in automations or conversational **Flows**.

If you need it later:

1. Generate a secret: `openssl rand -hex 32`
2. Add `AUTOMATION_CRON_SECRET` to Vercel environment variables
3. Redeploy
4. Ping these endpoints on a schedule (every 1–5 minutes) with header `x-cron-secret: <your-secret>`:
   - `GET https://your-project.vercel.app/api/automations/cron`
   - `GET https://your-project.vercel.app/api/flows/cron`

Free schedulers that work: [cron-job.org](https://cron-job.org), GitHub Actions, UptimeRobot.

---

## MVP limitations (Vercel free tier)

| Topic | Note |
|---|---|
| **Function timeout** | Hobby plan caps serverless functions at ~10 s. Most inbox traffic is fine; heavy media webhooks may need Vercel Pro (60 s — the app sets `maxDuration = 60` on the webhook route). |
| **Cron** | Vercel Cron requires Pro. Use a free external pinger for MVP (see above). |
| **Cold starts** | First request after idle can be slower (~1–2 s). Normal for serverless MVP. |
| **Rate limiting** | In-memory rate limits are per-instance, not global. Fine for MVP volume. |

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Build fails on Vercel | Missing env vars at build time | Add all required env vars; redeploy |
| Login redirect loop | Supabase auth URLs wrong | Match Site URL and Redirect URLs to your Vercel URL |
| Webhook verify fails | URL or token mismatch | Callback URL must be exact HTTPS path; verify token must match app + Meta |
| Webhook verify fails (502/timeout) | Deploy not ready | Wait for deploy to finish; check Vercel deployment logs |
| Messages not arriving | Wrong `META_APP_SECRET` | Match Meta App Secret in Vercel; redeploy |
| 500 after git pull | Missing migration | Run new files in `supabase/migrations/` in order |
| WhatsApp decrypt error | `ENCRYPTION_KEY` changed | Reset WhatsApp config in Settings and re-save |
| Env var change ignored | Old deployment cached | Redeploy after changing env vars |

Check runtime logs: Vercel → **Project → Logs** (filter by `/api/whatsapp/webhook` when debugging inbound messages).

---

## Security checklist (MVP)

- [ ] Never commit `.env.local` or paste service_role keys in client code
- [ ] `ENCRYPTION_KEY` stored only in Vercel env vars (not in git)
- [ ] Supabase RLS enabled (migrations handle this)
- [ ] Meta webhook uses HTTPS (Vercel provides this automatically)

---

## Related docs

- [local-setup.md](./local-setup.md) — local dev + detailed Supabase/WhatsApp setup
- [hostinger-vps-deployment.md](./hostinger-vps-deployment.md) — full VPS/Docker production path
- [wacrm.tech/docs/environment-variables](https://wacrm.tech/docs/environment-variables)
- [wacrm.tech/docs/troubleshooting](https://wacrm.tech/docs/troubleshooting)
