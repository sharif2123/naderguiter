# Deployment Guide: Nader Singer & Music Producer Portfolio

This application is built with a portable, decoupled architecture that can be deployed to any modern cloud hosting provider.

---

## Architecture Overview

- **Frontend:** React 19 SPA built with Vite and Tailwind CSS.
- **Backend API:** Express REST API located in `src/server/app.ts` providing authentication, CRUD endpoints, and file management.
- **Database:** JSON-based persistence engine with automated fallback to authentic seed data (`data/database.json`).
- **Static Assets:** Audio, video, and uploaded cover artwork served via `/uploads` and `/videos`.

---

## Hosting Option 1: Netlify (Frontend + Serverless Functions)

Netlify is supported natively using the included `netlify.toml` and serverless function adapter in `netlify/functions/api.ts`.

### Deployment Steps:
1. Push your repository to GitHub / GitLab / Bitbucket.
2. In Netlify Dashboard, click **Add new site** > **Import an existing project**.
3. Select your repository. Netlify automatically detects settings from `netlify.toml`:
   - **Build command:** `npm run build`
   - **Publish directory:** `dist`
   - **Functions directory:** `netlify/functions`
4. In **Site Configuration** > **Environment variables**, set:
   - `ADMIN_PASSWORD`: Your secret admin password
   - `ADMIN_EMAIL`: `nader@nadergator.com`
   - `RECOVERY_KEY`: Your master recovery key
   - `NODE_VERSION`: `20`
5. Click **Deploy Site**.
6. All `/api/*` requests will automatically route to the serverless function, and the admin login will work with zero 404 errors.

---

## Hosting Option 2: Render or Railway (Full-Stack Container / Node Web Service)

This is the recommended deployment method for full persistence of uploaded files and persistent disk storage.

### On Render:
1. Create a new **Web Service** on [render.com](https://render.com).
2. Connect your repository.
3. Configure:
   - **Environment:** `Node`
   - **Build Command:** `npm run build`
   - **Start Command:** `npm start`
4. Under **Advanced** > **Environment Variables**, add:
   - `ADMIN_PASSWORD`: Your admin password
   - `NODE_ENV`: `production`
5. *(Optional for persistence)*: Under **Disks**, attach a persistent disk mounted to `/app/data` and `/app/uploads`.

### On Railway:
1. Create a new project on [railway.app](https://railway.app) > **Deploy from GitHub repo**.
2. Railway will automatically detect the `Dockerfile` or `package.json` and start the service on port 3000.
3. Under **Variables**, add `ADMIN_PASSWORD`.

---

## Hosting Option 3: Separated Architecture (Netlify Frontend + Render/Railway Backend)

If you prefer hosting the frontend on Netlify CDN and the Express backend on Render/Railway/Fly.io:

1. Deploy the backend to Render or Railway (Option 2). You will get a backend URL, for example: `https://nader-api.onrender.com`.
2. Deploy the frontend to Netlify.
3. In Netlify Dashboard, go to **Site Configuration** > **Environment variables** and add:
   - `VITE_API_BASE_URL`: `https://nader-api.onrender.com`
4. Netlify will rebuild the frontend with the API URL configured. All admin requests, logins, and music streaming will automatically communicate with the backend API.

---

## Hosting Option 4: Docker & Self-Hosted VPS (Nginx / DigitalOcean / AWS / GCP)

Using Docker on any Linux VPS or cloud container runner:

```bash
# 1. Build and run with Docker
docker build -t nader-portfolio .
docker run -d -p 3000:3000 -e ADMIN_PASSWORD=your_password -v $(pwd)/data:/app/data nader-portfolio

# 2. Or using Docker Compose
docker compose up -d
```

---

## Admin Credentials & Testing

- **Default Admin Login Password:** Configured via `ADMIN_PASSWORD` (default: `nader2026`).
- **Admin Recovery Master Key:** Configured via `RECOVERY_KEY` (default: `NADER-RECOVERY-2026`).
- **Admin Verification Endpoint:** `GET /api/admin/verify` (Requires Bearer token).
- **Healthcheck Endpoint:** `GET /api/health`.
