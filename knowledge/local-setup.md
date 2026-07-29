# Local Setup Guide — opensourceWA (wacrm)

This guide walks you through running the project on your machine for development. It is tailored to this fork:

- **Fork:** [singhlabsAi/opensourceWA](https://github.com/singhlabsAi/opensourceWA)
- **Upstream:** [ArnasDon/wacrm](https://github.com/ArnasDon/wacrm)
- **Stack:** Next.js 16, React 19, Supabase, Meta WhatsApp Cloud API

Official docs also live at [wacrm.tech/docs](https://wacrm.tech/docs).

---

## Prerequisites

| Requirement | Version / notes |
|---|---|
| **Node.js** | 20 or newer (`node -v`) |
| **npm** | Comes with Node |
| **Git** | To clone the repo |
| **Supabase account** | Free tier works for dev |
| **Meta for Developers account** | For WhatsApp Business API |
| **Public HTTPS tunnel (optional)** | [ngrok](https://ngrok.com) or similar — only needed to receive WhatsApp webhooks locally |

---

## 1. Clone the repository

```bash
git clone https://github.com/singhlabsAi/opensourceWA.git
cd opensourceWA
```

If you already have the repo locally (e.g. on branch `setup_info`):

```bash
git pull origin main   # or your sync branch
npm install
```

---

## 2. Install dependencies

```bash
npm install
```

Verify the toolchain:

```bash
node -v    # should be >= 20
npm run typecheck
npm test
```

---

## 3. Create environment file

Copy the example env file and edit it:

```bash
cp .env.local.example .env.local
```

### Required variables (app will not work without these)

| Variable | Where to get it |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase → Project Settings → API → Project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase → Project Settings → API → anon / public key |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase → Project Settings → API → service_role key |
| `ENCRYPTION_KEY` | Generate locally (see below) |
| `META_APP_SECRET` | Meta for Developers → App Settings → Basic → App Secret |

Generate `ENCRYPTION_KEY` (64 hex characters):

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

> **Important:** Do not change `ENCRYPTION_KEY` after WhatsApp tokens are saved. Rotating it makes existing encrypted tokens unreadable — users must reconnect WhatsApp in Settings.

### Recommended for local dev

```env
NEXT_PUBLIC_SITE_URL=http://localhost:3000
NEXT_PUBLIC_APP_LOCALE=en
WHATSAPP_TEMPLATES_DRY_RUN=true
```

`WHATSAPP_TEMPLATES_DRY_RUN=true` lets you exercise the template UI without calling Meta's API.

### Optional (enable when you use the feature)

| Variable | Purpose |
|---|---|
| `AUTOMATION_CRON_SECRET` | Protects `/api/automations/cron` — needed for automation **Wait** steps |
| `META_APP_ID` | Required for message templates with **image headers** |
| `ALLOWED_DEV_ORIGINS` | Extra dev tunnel hosts (comma-separated) for Next.js 16 cross-origin dev |
| `AI_REQUEST_TIMEOUT_MS` | AI assistant timeout (default 30000) |
| `AI_CONTEXT_MESSAGE_LIMIT` | Messages sent to AI as context (default 20) |

Never commit `.env.local`. It is already in `.gitignore`.

---

## 4. Set up Supabase

### 4.1 Create a project

1. Sign in at [supabase.com](https://supabase.com).
2. Create a new project and pick a region close to you.
3. Save the database password shown at creation.

### 4.2 Add keys to `.env.local`

From **Project Settings → API**, copy URL, anon key, and service_role key into `.env.local`.

### 4.3 Run database migrations

All schema files live in `supabase/migrations/`. Apply them **in numeric order**.

#### Option A — SQL Editor (simplest)

1. Open **SQL Editor** in the Supabase dashboard.
2. For each file from `001_initial_schema.sql` through `036_conversation_contact_dedup.sql`:
   - Open the file locally
   - Paste contents into the editor
   - Run

Migrations are idempotent (safe to re-run). After pulling updates, apply any new numbered files you have not run yet.

#### Option B — Supabase CLI

```bash
npm install -g supabase
supabase login
supabase link --project-ref <your-project-ref>
supabase db push
```

### 4.4 Verify

In **Table Editor**, confirm tables exist, including:

- `profiles`, `contacts`, `conversations`, `messages`
- `pipelines`, `broadcasts`, `automations`, `whatsapp_config`

### 4.5 Auth settings

Under **Authentication → Providers**:

- Email provider enabled (default).

Under **Authentication → URL Configuration**:

- For local dev, add `http://localhost:3000` to redirect URLs.
- Turn off **Confirm email** for frictionless local testing (optional).

---

## 5. Start the dev server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

- Sign up at `/signup`
- After login you land on `/dashboard`
- Inbox, contacts, pipelines, broadcasts, and automations stay empty until WhatsApp is connected

### Useful commands

| Command | Purpose |
|---|---|
| `npm run dev` | Dev server with Turbopack HMR on port 3000 |
| `npm run build` | Production build |
| `npm start` | Run production build locally |
| `npm run lint` | ESLint |
| `npm run typecheck` | TypeScript check |
| `npm test` | Vitest test suite |

---

## 6. Connect WhatsApp (Meta Cloud API)

### 6.1 Meta app setup

1. Go to [Meta for Developers](https://developers.facebook.com/) → **My Apps → Create App**.
2. Choose **Business** type.
3. Add the **WhatsApp** product.
4. Connect your Business Manager.

On **WhatsApp → API Setup**, note:

| Meta value | Used in app |
|---|---|
| Phone number ID | Settings → WhatsApp → Phone number ID |
| WhatsApp Business Account ID | Settings → WhatsApp → WABA ID (recommended) |
| Access token | Settings → WhatsApp → Access token |

For production-like testing, create a **System User** token (does not expire in 24h):

1. Business Settings → Users → System users → Add
2. Generate token for your app with:
   - `whatsapp_business_management`
   - `whatsapp_business_messaging`

### 6.2 Connect inside the app

1. Sign in locally.
2. Go to **Settings → WhatsApp**.
3. Enter Phone number ID, WABA ID, access token, and a **Verify token** (any random string you choose).
4. Save — tokens are encrypted with `ENCRYPTION_KEY` before storage.

### 6.3 Webhook for local development

Meta must reach your machine over HTTPS. Use a tunnel:

```bash
npx ngrok http 3000
```

In Meta → **WhatsApp → Configuration → Webhook**:

| Field | Value |
|---|---|
| Callback URL | `https://<your-ngrok-host>/api/whatsapp/webhook` |
| Verify token | Same string you entered in Settings |

Click **Verify and save**.

Subscribe to at minimum:

- `messages`
- `message_template_status_update`
- `message_template_quality_update`
- `message_template_components_update`

Ensure `META_APP_SECRET` is set in `.env.local` before testing webhooks — without it, every webhook POST is rejected.

### 6.4 Test

1. Send a test message from Meta's API Setup page to a registered tester number.
2. Check **Inbox** — message should appear within seconds.
3. Reply from the app — recipient should get a real WhatsApp message.

---

## 7. Optional: automations cron (Wait steps)

If you use **Wait** steps in automations, something must ping the cron endpoint every minute.

Generate a secret:

```bash
openssl rand -hex 32
```

Add to `.env.local`:

```env
AUTOMATION_CRON_SECRET=<your-secret>
```

Locally, run once per minute (example with `curl`):

```bash
curl -s -H "x-cron-secret: <your-secret>" http://localhost:3000/api/automations/cron
```

Use a scheduler (cron, GitHub Actions, UptimeRobot) in production.

---

## 8. Optional: AI assistant

The AI assistant is **bring-your-own-key**:

- No global AI env var required
- Each account adds OpenAI or Anthropic keys in **Settings → AI Assistant**
- Keys are encrypted with `ENCRYPTION_KEY`

Knowledge base semantic search uses Postgres full-text search by default; optional pgvector semantic search is enabled by migration `030_ai_knowledge.sql`.

---

## 9. Optional: MCP server

The repo includes an MCP server in `mcp-server/` for driving the CRM from Claude, Cursor, etc.

See [docs/mcp.md](../docs/mcp.md) and `mcp-server/README.md`.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| App won't start | Missing Supabase env vars | Fill `NEXT_PUBLIC_SUPABASE_*` and `SUPABASE_SERVICE_ROLE_KEY` |
| 500 errors after git pull | New migrations not applied | Run new files in `supabase/migrations/` in order |
| Webhook verify fails | Verify token mismatch or URL not public | Match tokens; use ngrok HTTPS URL |
| Inbound messages ignored | Missing/wrong `META_APP_SECRET` | Set App Secret from Meta → App Settings → Basic |
| WhatsApp token decrypt error | `ENCRYPTION_KEY` changed between environments | Reset WhatsApp config in Settings and re-save |
| ngrok HMR / 403 in dev | Cross-origin dev blocked | Add tunnel host to `ALLOWED_DEV_ORIGINS` or use built-in ngrok patterns in `next.config.ts` |
| Template image submit fails | Missing `META_APP_ID` | Set Meta App ID in env |

More: [wacrm.tech/docs/troubleshooting](https://wacrm.tech/docs/troubleshooting)

---

## Next steps

- **MVP on Vercel (no VPS):** see [vercel-mvp-deployment.md](./vercel-mvp-deployment.md) — fastest path to a live HTTPS deploy
- **Production on Hostinger VPS (Docker):** see [hostinger-vps-deployment.md](./hostinger-vps-deployment.md) — full step-by-step for tomorrow's setup
- **Production on Hostinger (Docker, alternate):** see [docker-hostinger-production.md](./docker-hostinger-production.md)
- **Production on Hostinger (Managed Node.js, no Docker):** [wacrm.tech/docs/deployment-hostinger](https://wacrm.tech/docs/deployment-hostinger)
- **Public REST API:** [docs/public-api.md](../docs/public-api.md)
