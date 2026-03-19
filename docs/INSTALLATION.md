# Installation & Deployment Guide

[← Back to README](README.md) · [Architecture](ARCHITECTURE.md) · [API Services](API_SERVICES.md) · [Components](COMPONENTS.md) · [User Guide](USER_GUIDE.md)

> **Audience:** System administrators and DevOps engineers deploying the application, and developers setting up a local environment.

This guide covers local development setup, production build, nginx virtual-host configuration, post-deploy configuration management, and common troubleshooting steps.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Local Development Setup](#local-development-setup)
3. [Environment Configuration](#environment-configuration)
4. [Production Build](#production-build)
5. [Nginx Deployment](#nginx-deployment)
6. [Configuration After Deployment](#configuration-after-deployment)
7. [Updating an Existing Deployment](#updating-an-existing-deployment)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Development machine

| Requirement | Version | Notes |
|-------------|---------|-------|
| Node.js | ≥ 18 LTS | [nodejs.org](https://nodejs.org/) |
| npm | ≥ 9 | Ships with Node.js |
| Git | any | For cloning / version control |

### Production server

| Requirement | Notes |
|-------------|-------|
| Linux (Debian/Ubuntu recommended) | Any distribution that runs nginx |
| nginx | Any recent stable version |
| Backend API server | Flask application reachable from the frontend server |

---

## Local Development Setup

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd bollettino_cfd_frontend
```

### 2. Install dependencies

```bash
npm install
```

### 3. Configure server URLs for development

Create or edit `public/config.json`:

```json
{
  "LOGIN_URL": "http://localhost:5000",
  "SERVER_URL": "http://localhost:5000",
  "WS_URL":    "ws://localhost:5000"
}
```

> **Note:** `public/config.json` is served as a static file and is read at **runtime** in the browser. It is not bundled into the JavaScript. This means one build artifact can be repointed to a different backend simply by editing this file.

### 4. Start the dev server

```bash
npm run dev
```

Vite will print the local URL (typically `http://localhost:5173`). The dev server includes:
- Hot Module Replacement (HMR) for instant updates
- Alias `@` → `src/`
- `base: './'` (relative paths, suitable for sub-directory deployments)

### 5. Lint and format (optional)

```bash
npm run lint      # ESLint with auto-fix
npm run format    # Prettier formatting on src/
```

---

## Environment Configuration

### How server URLs are resolved

The URL resolution chain is:

```
Browser start
  └─► LoginView fetches /public/config.json
        └─► Stores LOGIN_URL, SERVER_URL, WS_URL in localStorage
              └─► service.js reads from localStorage at module load time
                    └─► All Axios calls use SERVER_URL / LOGIN_URL
```

### Config files reference

| File | Purpose | Loaded by |
|------|---------|-----------|
| `public/config.json` | **Primary runtime config** — always used in production | Browser at startup |
| `src/assets/config_local.json` | Template for local development (not loaded automatically) | Manual copy to `public/config.json` |
| `src/assets/config_remote.json` | Template for staging/production (not loaded automatically) | Manual copy to `public/config.json` |

### Config file format

```json
{
  "LOGIN_URL": "http://<backend-host>:<port>",
  "SERVER_URL": "http://<backend-host>:<port>",
  "WS_URL":    "ws://<backend-host>:<port>"
}
```

> If the backend uses HTTPS/WSS in production, update the schemes accordingly:
> `"https://..."` and `"wss://..."`.

---

## Production Build

### 1. Set the correct config

Before building, ensure `public/config.json` contains the **production** server URLs, or plan to replace it after deployment (recommended — see step 6 below).

### 2. Build

```bash
npm run build
```

This command:
1. Runs `vue-tsc --noEmit` for TypeScript type-checking.
2. Runs `vite build` to produce the `dist/` directory.

`dist/` will contain:
```
dist/
├── index.html
├── config.json       ← copied from public/
├── assets/
│   ├── index-<hash>.js
│   ├── index-<hash>.css
│   └── ... (other chunks)
└── img/
    └── ...
```

The `base: './'` setting in `vite.config.ts` ensures all asset paths are relative, so the app can be served from any sub-directory.

### 3. Preview the build locally

```bash
npm run preview
```

Opens the production build on `http://localhost:4173`.

---

## Nginx Deployment

### Step 1 — Create the web root directory

```bash
sudo mkdir -p /var/www/html
```

Or a custom path such as `/var/www/multirischio`:

```bash
sudo mkdir -p /var/www/multirischio
```

### Step 2 — Copy the nginx site configuration

The nginx configuration template is located at `documents/nginx/multirischio_front`.

```bash
sudo cp documents/nginx/multirischio_front /etc/nginx/sites-available/multirischio_front
```

### Step 3 — Review and edit the nginx configuration

Open the file and verify the settings:

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;    # ← change to your web root
    index index.html;

    server_name _;         # ← replace with your hostname if needed

    location / {
        # SPA routing: fall back to index.html for all paths
        try_files $uri $uri/ /index.html =404;
    }

    location @rewrites {
        rewrite ^(.+)$ /index.html last;
    }

    # Cache static assets aggressively
    location ~* \.(?:ico|css|js|gif|jpe?g|png)$ {
        expires max;
        add_header Pragma public;
        add_header Cache-Control "public, must-revalidate, proxy-revalidate";
    }
}
```

> **Important:** The `try_files $uri $uri/ /index.html` directive is required for Vue Router's HTML5 history mode. Without it, direct URL navigation (e.g. `/meteo`) returns 404.

### Step 4 — Enable the site

```bash
sudo ln -s /etc/nginx/sites-available/multirischio_front \
           /etc/nginx/sites-enabled/multirischio_front
```

If a default site conflicts, disable it:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

### Step 5 — Validate and reload nginx

```bash
sudo nginx -t          # test configuration syntax
sudo nginx -s reload   # reload without downtime
# or, if reload doesn't pick up changes:
sudo systemctl restart nginx
```

### Step 6 — Copy the dist files

```bash
sudo cp -r dist/* /var/www/html/
```

Or if using a custom directory:

```bash
sudo cp -r dist/* /var/www/multirischio/
```

### Step 7 — Update config.json for production

After copying, edit the runtime configuration file to point to the production backend:

```bash
sudo nano /var/www/html/config.json
```

```json
{
  "LOGIN_URL": "http://your-production-backend:5000",
  "SERVER_URL": "http://your-production-backend:5000",
  "WS_URL":    "ws://your-production-backend:5000"
}
```

---

## Configuration After Deployment

`config.json` is intentionally kept separate from the build bundle. This means:

- You can **switch backend servers without rebuilding** the frontend.
- CI/CD pipelines can inject environment-specific configs as a post-build step.
- The file must remain accessible at `/config.json` relative to the site root.

### Permissions

Ensure the web root is readable by the nginx worker process:

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

---

## Updating an Existing Deployment

1. Pull latest code and rebuild:
   ```bash
   git pull
   npm install          # in case dependencies changed
   npm run build
   ```

2. Clear and replace the web root contents:
   ```bash
   sudo rm -rf /var/www/html/*
   sudo cp -r dist/* /var/www/html/
   ```

3. **Re-apply** `config.json` (it is overwritten by the copy):
   ```bash
   sudo nano /var/www/html/config.json
   # restore production URLs
   ```

> **Tip:** Keep a backup of the production `config.json` outside the web root (e.g. `/etc/multirischio/config.json`) and copy it in as part of your deploy script.

---

## Troubleshooting

### Blank page / 404 on direct URL access

The nginx `try_files` directive is not in place. Ensure the `location /` block includes:

```nginx
try_files $uri $uri/ /index.html =404;
```

### Login fails immediately — "CORS error" in browser console

The backend is not sending CORS headers that allow the frontend origin. Configure CORS on the Flask backend to allow `http://<frontend-host>`.

### Maps not loading / grey tiles

Leaflet tile layers may be blocked. Check the browser console; if tile requests fail, verify that the backend or tile server is reachable from the browser.

### "Credenziali scadute" toast appears immediately

The `auth_token` in `localStorage` has expired or is malformed. Log out (clear `localStorage`) and log back in.

### Socket.IO not connecting

Verify that `WS_URL` in `config.json` is set correctly and that the backend WebSocket server is running and accessible from the browser. Check browser DevTools → Network → WS tab.

### Build fails with TypeScript errors

Run:
```bash
npm run type-check
```
Fix reported errors before running `npm run build`. Alternatively, use `npm run build-only` to skip type-checking (not recommended for production).

---

[↑ Back to top](#installation--deployment-guide) · [← Back to README](README.md)
