# Hostinger VPS Deployment Guide — opensourceWA (wacrm)

Complete step-by-step guide to deploy this WhatsApp CRM on a **Hostinger VPS** with Docker, Nginx, and Let's Encrypt SSL.

**Estimated time:** 1–2 hours (first time)

**Stack:**

- VPS: Hostinger Ubuntu 22.04/24.04 (2 GB+ RAM)
- App: Next.js 16 (Docker container)
- Database: Supabase (hosted — NOT on VPS)
- WhatsApp: Meta Cloud API webhooks

---

## Pre-flight checklist (do before SSH)

Gather these before you start:

| Item | Where to get it |
|------|-----------------|
| VPS IP address | Hostinger hPanel → VPS |
| Domain/subdomain | e.g. `crm.yourdomain.com` |
| Supabase project URL | Supabase → Settings → API |
| Supabase anon key | Supabase → Settings → API |
| Supabase service_role key | Supabase → Settings → API |
| Meta App Secret | Meta Developers → App Settings → Basic |
| Meta App ID (optional) | Meta Developers → App Settings → Basic |
| WhatsApp Phone Number ID | Meta → WhatsApp → API Setup |
| WhatsApp Business Account ID | Meta → WhatsApp → API Setup |
| WhatsApp Access Token | Meta System User token (recommended for prod) |

Generate secrets on your laptop:

```bash
# ENCRYPTION_KEY (64 hex chars)
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# AUTOMATION_CRON_SECRET
openssl rand -hex 32

# Webhook verify token (any random string you choose)
openssl rand -hex 16
```

> **Important:** Save `ENCRYPTION_KEY` permanently. Changing it breaks stored WhatsApp tokens.

---

## Part 1 — Supabase setup

### 1.1 Create Supabase project

1. Go to [supabase.com](https://supabase.com) → New project
2. Pick a region close to your VPS/users
3. Save the database password

### 1.2 Run database migrations

All migrations are in `supabase/migrations/`. Apply **in numeric order** (`001` through `036`).

**Option A — SQL Editor (easiest):**

1. Supabase dashboard → **SQL Editor**
2. Open each file locally, paste, run
3. Repeat for all 36 files

**Option B — Supabase CLI:**

```bash
npm install -g supabase
supabase login
supabase link --project-ref <your-project-ref>
supabase db push
```

### 1.3 Verify tables

In **Table Editor**, confirm these exist:

- `profiles`, `contacts`, `conversations`, `messages`
- `pipelines`, `broadcasts`, `automations`, `whatsapp_config`

### 1.4 Auth settings (for production)

Supabase → **Authentication → URL Configuration**:

- Site URL: `https://crm.yourdomain.com` (set after DNS is live)
- Redirect URLs: `https://crm.yourdomain.com/**`

---

## Part 2 — VPS preparation

### 2.1 DNS

Hostinger **hPanel → Domains → DNS**:

| Type | Name | Value |
|------|------|-------|
| A | `crm` (or `@`) | `<YOUR_VPS_IP>` |

Wait 5–30 minutes for propagation. Verify:

```bash
dig +short crm.yourdomain.com
```

### 2.2 SSH into VPS

```bash
ssh root@<YOUR_VPS_IP>
```

### 2.3 System update + install tools

```bash
apt update && apt upgrade -y
apt install -y ca-certificates curl git nginx certbot python3-certbot-nginx ufw
```

### 2.4 Install Docker

```bash
curl -fsSL https://get.docker.com | sh
systemctl enable docker
systemctl start docker
docker --version
docker compose version
```

### 2.5 Firewall

```bash
ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
ufw status
```

---

## Part 3 — Docker files (already in repo)

The following files are at the repository root — no need to create them manually:

- `Dockerfile`
- `docker-compose.yml`
- `.dockerignore`

Ensure these are pushed to your GitHub fork before cloning on the VPS.

---

## Part 4 — Deploy on VPS

### 4.1 Clone the repo

```bash
mkdir -p /opt/wacrm
cd /opt/wacrm
git clone https://github.com/singhlabsAi/opensourceWA.git .
# Or your fork URL:
# git clone https://github.com/<your-username>/opensourceWA.git .
```

### 4.2 Create production env file

```bash
nano /opt/wacrm/.env.production
```

Paste (replace all placeholder values):

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# Security
ENCRYPTION_KEY=your-64-char-hex-key
META_APP_SECRET=your-meta-app-secret

# Public URL (no trailing slash)
NEXT_PUBLIC_SITE_URL=https://crm.yourdomain.com
NEXT_PUBLIC_APP_LOCALE=en

# Recommended
META_APP_ID=your-meta-app-id
AUTOMATION_CRON_SECRET=your-long-random-secret

# Optional — harden invite URLs on bare VPS
ALLOWED_INVITE_HOSTS=crm.yourdomain.com
```

Secure the file:

```bash
chmod 600 /opt/wacrm/.env.production
```

### 4.3 Build and start

```bash
cd /opt/wacrm
docker compose build --no-cache
docker compose up -d
docker compose logs -f wacrm
```

Press `Ctrl+C` to stop following logs.

### 4.4 Verify app is running

```bash
docker compose ps
curl -I http://127.0.0.1:3000
```

Expected: HTTP 200 or 307 redirect to login.

---

## Part 5 — Nginx + SSL

### 5.1 Create Nginx config

```bash
nano /etc/nginx/sites-available/wacrm
```

```nginx
server {
    listen 80;
    server_name crm.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }
}
```

Enable the site:

```bash
ln -s /etc/nginx/sites-available/wacrm /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

### 5.2 Get SSL certificate

```bash
certbot --nginx -d crm.yourdomain.com
```

Follow prompts (enter email, agree to terms). Certbot auto-configures HTTPS.

Verify in browser: `https://crm.yourdomain.com`

---

## Part 6 — Post-deploy configuration

### 6.1 Supabase auth URLs

Supabase → **Authentication → URL Configuration**:

- Site URL: `https://crm.yourdomain.com`
- Redirect URLs: `https://crm.yourdomain.com/**`

### 6.2 Meta WhatsApp webhook

Meta for Developers → **WhatsApp → Configuration**:

| Field | Value |
|-------|-------|
| Callback URL | `https://crm.yourdomain.com/api/whatsapp/webhook` |
| Verify token | Your chosen verify token string |

Click **Verify and save**.

Subscribe to webhook fields:

- `messages`
- `message_template_status_update`
- `message_template_quality_update`
- `message_template_components_update`

Ensure `META_APP_SECRET` in `.env.production` matches Meta → App Settings → Basic.

### 6.3 Connect WhatsApp in the app

1. Open `https://crm.yourdomain.com`
2. Sign up / log in
3. Go to **Settings → WhatsApp**
4. Enter:
   - Phone number ID
   - WABA ID
   - Access token
   - Verify token (same as Meta webhook)
5. Save

### 6.4 Automations cron (if using Wait steps)

```bash
crontab -e
```

Add:

```cron
* * * * * curl -s -H "x-cron-secret: YOUR_AUTOMATION_CRON_SECRET" https://crm.yourdomain.com/api/automations/cron > /dev/null 2>&1
```

---

## Part 7 — Smoke test

- [ ] Site loads at `https://crm.yourdomain.com`
- [ ] Can sign up / log in
- [ ] Dashboard loads without 500 errors
- [ ] WhatsApp settings save successfully
- [ ] Meta webhook shows green checkmark (verified)
- [ ] Send test message from Meta → appears in Inbox
- [ ] Reply from app → recipient gets WhatsApp message

---

## Part 8 — Deploying updates

```bash
cd /opt/wacrm
git pull origin main

# Apply new Supabase migrations FIRST (if any)
# supabase/migrations/*.sql in numeric order via Supabase SQL Editor

docker compose build
docker compose up -d
docker compose logs -f wacrm
```

---

## Operations reference

| Task | Command |
|------|---------|
| View logs | `docker compose logs -f wacrm` |
| Restart app | `docker compose restart wacrm` |
| Stop app | `docker compose down` |
| Container status | `docker compose ps` |
| Disk usage | `docker system df` |
| Clean old images | `docker image prune -f` |
| Nginx logs | `tail -f /var/log/nginx/error.log` |
| Test SSL renewal | `certbot renew --dry-run` |

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Build fails / killed | VPS RAM too low | Upgrade to 2 GB+ or add swap (see below) |
| 502 Bad Gateway | Container not running | `docker compose ps` + check logs |
| Webhook verify fails | SSL or URL wrong | Ensure HTTPS works; URL must match exactly |
| Messages not arriving | Wrong `META_APP_SECRET` | Match Meta App Secret; restart container |
| 500 after deploy | Missing migration | Run new SQL files in `supabase/migrations/` |
| WhatsApp decrypt error | `ENCRYPTION_KEY` changed | Re-save WhatsApp settings in app |
| Invite links wrong host | Spoofed Host header | Set `ALLOWED_INVITE_HOSTS` and `NEXT_PUBLIC_SITE_URL` |

### Add swap if build runs out of memory (1 GB VPS)

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
free -h
```

---

## Security checklist

- [ ] `chmod 600 .env.production`
- [ ] UFW enabled (SSH + Nginx only)
- [ ] SSH key auth (disable password login)
- [ ] `ALLOWED_INVITE_HOSTS` set
- [ ] Regular `apt upgrade` on VPS
- [ ] Supabase DB backups enabled (paid plan)

---

## Related docs

- [local-setup.md](./local-setup.md) — local dev + Supabase/WhatsApp setup
- [docker-hostinger-production.md](./docker-hostinger-production.md) — alternate Docker-focused reference
- [wacrm.tech/docs/environment-variables](https://wacrm.tech/docs/environment-variables)
- [wacrm.tech/docs/troubleshooting](https://wacrm.tech/docs/troubleshooting)
- [docs/public-api.md](../docs/public-api.md) — REST API
