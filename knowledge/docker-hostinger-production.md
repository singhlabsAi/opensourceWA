# Docker Production Deployment on Hostinger VPS

This guide deploys **opensourceWA (wacrm)** as a Docker container on a **Hostinger VPS**. Use this path when you want full control over the runtime, repeatable builds, and container-based ops.

> **Note:** Hostinger **Managed Node.js** hosting (hPanel Git deploy, no Docker) is the simpler default for this template. See [wacrm.tech/docs/deployment-hostinger](https://wacrm.tech/docs/deployment-hostinger). Use Docker on a **VPS** when you need custom process layout, sidecars, or infrastructure-as-code.

**Assumptions:**

- Supabase project is created and migrations are applied (see [local-setup.md](./local-setup.md))
- Meta WhatsApp app and webhook credentials are ready
- You have a Hostinger VPS with root/sudo access
- Domain points to the VPS (e.g. `crm.example.com`)

---

## Architecture

```text
Internet
   │
   ▼
[Nginx + Let's Encrypt]  ← Hostinger VPS
   │  :443 → :3000
   ▼
[Docker: wacrm app]      ← Next.js production server
   │
   ├── Supabase (hosted)   ← Postgres, Auth, Storage
   └── Meta Cloud API      ← WhatsApp webhooks + outbound messages
```

The app container does **not** include Postgres. Database stays on Supabase (recommended).

---

## 1. VPS preparation (Hostinger)

### 1.1 Provision the VPS

1. Purchase a Hostinger VPS plan with at least **2 GB RAM** (`npm run build` is memory-heavy).
2. Choose Ubuntu 22.04 or 24.04.
3. Note the VPS IP address.

### 1.2 Point your domain

In Hostinger **hPanel → Domains → DNS**:

| Type | Name | Value |
|---|---|---|
| A | `crm` (or `@`) | `<VPS IP>` |

Wait for DNS propagation (minutes to a few hours).

### 1.3 SSH into the server

```bash
ssh root@<vps-ip>
```

### 1.4 Install Docker

```bash
apt update && apt upgrade -y
apt install -y ca-certificates curl git

curl -fsSL https://get.docker.com | sh
systemctl enable docker
systemctl start docker

# Optional: run docker without sudo
usermod -aG docker $USER
```

Verify:

```bash
docker --version
docker compose version
```

### 1.5 Install Nginx and Certbot

```bash
apt install -y nginx certbot python3-certbot-nginx
systemctl enable nginx
```

---

## 2. Add Docker files to the project

Create these files at the **repository root** (same level as `package.json`).

### 2.1 `.dockerignore`

```dockerignore
.git
.github
node_modules
.next
.env*
!.env.local.example
npm-debug.log*
.DS_Store
coverage
.vscode
.idea
knowledge
mcp-server/node_modules
```

### 2.2 `Dockerfile`

Multi-stage build optimized for Next.js production:

```dockerfile
# syntax=docker/dockerfile:1

FROM node:20-alpine AS base
WORKDIR /app
ENV NEXT_TELEMETRY_DISABLED=1

FROM base AS deps
RUN apk add --no-cache libc6-compat
COPY package.json package-lock.json ./
RUN npm ci

FROM base AS builder
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# NEXT_PUBLIC_* must be available at build time
ARG NEXT_PUBLIC_SUPABASE_URL
ARG NEXT_PUBLIC_SUPABASE_ANON_KEY
ARG NEXT_PUBLIC_SITE_URL
ARG NEXT_PUBLIC_APP_LOCALE=en

ENV NEXT_PUBLIC_SUPABASE_URL=$NEXT_PUBLIC_SUPABASE_URL
ENV NEXT_PUBLIC_SUPABASE_ANON_KEY=$NEXT_PUBLIC_SUPABASE_ANON_KEY
ENV NEXT_PUBLIC_SITE_URL=$NEXT_PUBLIC_SITE_URL
ENV NEXT_PUBLIC_APP_LOCALE=$NEXT_PUBLIC_APP_LOCALE

# Placeholders for modules loaded at build time
ENV ENCRYPTION_KEY=0000000000000000000000000000000000000000000000000000000000000000
ENV META_APP_SECRET=build-time-placeholder

RUN npm run build

FROM base AS runner
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs \
  && adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/.next ./.next

USER nextjs
EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME=0.0.0.0

CMD ["npm", "start"]
```

### 2.3 `docker-compose.yml`

```yaml
services:
  wacrm:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        NEXT_PUBLIC_SUPABASE_URL: ${NEXT_PUBLIC_SUPABASE_URL}
        NEXT_PUBLIC_SUPABASE_ANON_KEY: ${NEXT_PUBLIC_SUPABASE_ANON_KEY}
        NEXT_PUBLIC_SITE_URL: ${NEXT_PUBLIC_SITE_URL}
        NEXT_PUBLIC_APP_LOCALE: ${NEXT_PUBLIC_APP_LOCALE:-en}
    container_name: wacrm
    restart: unless-stopped
    ports:
      - "127.0.0.1:3000:3000"
    env_file:
      - .env.production
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://127.0.0.1:3000/"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 40s
```

Binding to `127.0.0.1:3000` keeps the app off the public internet — Nginx terminates TLS and proxies to it.

---

## 3. Production environment file

On the VPS, clone your fork:

```bash
mkdir -p /opt/wacrm
cd /opt/wacrm
git clone https://github.com/singhlabsAi/opensourceWA.git .
```

Create `/opt/wacrm/.env.production` (never commit this file):

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# Security
ENCRYPTION_KEY=your-64-char-hex-key
META_APP_SECRET=your-meta-app-secret

# Public URL (no trailing slash)
NEXT_PUBLIC_SITE_URL=https://crm.example.com
NEXT_PUBLIC_APP_LOCALE=en

# Recommended for production
META_APP_ID=your-meta-app-id
AUTOMATION_CRON_SECRET=your-long-random-secret

# Optional hardening for invite URLs on bare VPS
# ALLOWED_INVITE_HOSTS=crm.example.com
```

Generate secrets on the VPS:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"   # ENCRYPTION_KEY
openssl rand -hex 32                                                          # AUTOMATION_CRON_SECRET
```

> Use the **same** `ENCRYPTION_KEY` across redeploys. Changing it breaks stored WhatsApp tokens.

---

## 4. Build and run the container

```bash
cd /opt/wacrm
docker compose build --no-cache
docker compose up -d
docker compose logs -f wacrm
```

Check health:

```bash
curl -I http://127.0.0.1:3000
```

You should get an HTTP response (redirect to login is fine).

---

## 5. Nginx reverse proxy + SSL

### 5.1 Nginx site config

Create `/etc/nginx/sites-available/wacrm`:

```nginx
server {
    listen 80;
    server_name crm.example.com;

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

Enable and test:

```bash
ln -s /etc/nginx/sites-available/wacrm /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

### 5.2 Let's Encrypt certificate

```bash
certbot --nginx -d crm.example.com
```

Certbot configures HTTPS and auto-renewal.

Verify in browser: `https://crm.example.com`

---

## 6. Supabase auth redirect URLs

In Supabase → **Authentication → URL Configuration**, add:

- Site URL: `https://crm.example.com`
- Redirect URLs: `https://crm.example.com/**`

---

## 7. Meta WhatsApp webhook (production)

In Meta for Developers → **WhatsApp → Configuration**:

| Field | Value |
|---|---|
| Callback URL | `https://crm.example.com/api/whatsapp/webhook` |
| Verify token | Same value saved in app Settings → WhatsApp |

Subscribe to:

- `messages`
- `message_template_status_update`
- `message_template_quality_update`
- `message_template_components_update`

Ensure `META_APP_SECRET` in `.env.production` matches Meta → App Settings → Basic.

---

## 8. Automations cron (if using Wait steps)

Add a system cron job on the VPS:

```bash
crontab -e
```

```cron
* * * * * curl -s -H "x-cron-secret: YOUR_AUTOMATION_CRON_SECRET" https://crm.example.com/api/automations/cron > /dev/null 2>&1
```

Replace the secret with the value from `.env.production`.

---

## 9. Deploying updates

```bash
cd /opt/wacrm
git pull origin main

# Apply any new Supabase migrations FIRST (Supabase SQL Editor or CLI)
# Files: supabase/migrations/*.sql in numeric order

docker compose build
docker compose up -d
docker compose logs -f wacrm
```

### Zero-downtime tip

For larger teams, tag images and swap containers:

```bash
docker compose build
docker compose up -d --no-deps wacrm
```

---

## 10. Operations checklist

| Task | Command / location |
|---|---|
| View logs | `docker compose logs -f wacrm` |
| Restart app | `docker compose restart wacrm` |
| Stop app | `docker compose down` |
| Disk usage | `docker system df` |
| Prune old images | `docker image prune -f` |
| Nginx logs | `/var/log/nginx/access.log`, `error.log` |
| SSL renewal | `certbot renew --dry-run` |

---

## 11. Security hardening (recommended)

1. **Firewall** — allow only SSH (22), HTTP (80), HTTPS (443):

   ```bash
   ufw allow OpenSSH
   ufw allow 'Nginx Full'
   ufw enable
   ```

2. **Secrets** — keep `.env.production` off Git; restrict permissions:

   ```bash
   chmod 600 /opt/wacrm/.env.production
   ```

3. **SSH** — disable password auth; use SSH keys only.

4. **Updates** — run `apt upgrade` regularly on the VPS.

5. **Invite URLs** — set `ALLOWED_INVITE_HOSTS` or `NEXT_PUBLIC_SITE_URL` on bare VPS deploys.

6. **Backups** — Supabase handles DB backups on paid plans; also snapshot VPS via Hostinger panel.

---

## 12. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Build fails OOM | VPS RAM too low | Upgrade VPS or add 2 GB swap |
| Blank page / 502 | Container not running | `docker compose ps` and check logs |
| Webhook verify fails | Nginx or SSL misconfig | Ensure HTTPS works; callback URL exact |
| Messages not arriving | Wrong `META_APP_SECRET` | Match Meta App Secret; restart container |
| 500 after deploy | Missing DB migration | Apply new `supabase/migrations/` files |
| Stale assets after deploy | CDN/browser cache | Hard refresh; check Cache-Control headers |
| WhatsApp decrypt error | `ENCRYPTION_KEY` changed | Re-save WhatsApp settings in app |

---

## Alternative: Hostinger Managed Node.js (no Docker)

If you do not need Docker, Hostinger's managed path is faster:

1. hPanel → Websites → Create → **Node.js**
2. Connect GitHub fork → branch `main`
3. Set env vars in hPanel
4. Run `npm ci && npm run build`
5. Start with `npm start`

Full guide: [wacrm.tech/docs/deployment-hostinger](https://wacrm.tech/docs/deployment-hostinger)

---

## Related docs

- [local-setup.md](./local-setup.md) — dev environment and Supabase/WhatsApp setup
- [wacrm.tech/docs/environment-variables](https://wacrm.tech/docs/environment-variables)
- [docs/public-api.md](../docs/public-api.md) — REST API keys
- [docs/mcp.md](../docs/mcp.md) — MCP server (runs separately, not in this container)
