# Railway Deployment Guide

This project is split into two services: **backend** (Express + Prisma) and **frontend** (Next.js).
Deploy them as two separate Railway services connected to one Postgres database.

## Architecture on Railway
```
[Railway Postgres Plugin] ──> [Backend Service] <── [Frontend Service]
```

---

## Step 1: Create a Railway Account & Project
1. Go to https://railway.app and sign up (GitHub login recommended).
2. Click **New Project** → **Empty Project**.
3. Name it (e.g. `import-export-website`).

---

## Step 2: Add a Postgres Database
1. Inside your project, click **+ New** → **Database** → **Add PostgreSQL**.
2. Railway will provision a Postgres instance and automatically create a `DATABASE_URL` variable.

---

## Step 3: Deploy the Backend
1. Click **+ New** → **GitHub Repo** (connect your GitHub account if not done).
2. Select your repository.
3. Set the **Root Directory** to `/backend`.
4. Railway will auto-detect Node.js and run `npm install && npm run build`, then `npm run start`.

### Backend Environment Variables
Go to the backend service → **Variables** tab and add:

| Variable | Value |
|---|---|
| `DATABASE_URL` | Click "Add Reference" → select Postgres → `DATABASE_URL` |
| `JWT_SECRET` | Generate: `openssl rand -base64 32` |
| `FRONTEND_ORIGIN` | Your frontend Railway URL (set after frontend is deployed) |
| `SENDGRID_API_KEY` | *(optional)* Your SendGrid API key |
| `SENDGRID_FROM_EMAIL` | *(optional)* Verified sender email |
| `ADMIN_NOTIFICATION_EMAIL` | *(optional)* Your admin email |
| `AWS_ACCESS_KEY_ID` | *(optional)* For S3 file storage |
| `AWS_SECRET_ACCESS_KEY` | *(optional)* For S3 file storage |
| `AWS_S3_BUCKET` | *(optional)* Your S3 bucket name |
| `AWS_S3_REGION` | *(optional, default: ap-south-1)* |

5. After deploy, note the backend's public URL (e.g. `https://backend-xyz.up.railway.app`).

---

## Step 4: Deploy the Frontend
1. Click **+ New** → **GitHub Repo** → same repository.
2. Set the **Root Directory** to `/frontend`.

### Frontend Environment Variables
| Variable | Value |
|---|---|
| `NEXT_PUBLIC_API_BASE_URL` | `https://your-backend.up.railway.app/api` |
| `NEXT_PUBLIC_SITE_URL` | `https://your-frontend.up.railway.app` |

3. After deploy, note the frontend's public URL.

---

## Step 5: Update FRONTEND_ORIGIN on Backend
Go back to the **backend** service → **Variables** and update:
- `FRONTEND_ORIGIN` = `https://your-frontend.up.railway.app`

This will trigger a redeploy automatically.

---

## Step 6: Run Database Migrations
Migrations run automatically on every deploy via `prisma migrate deploy` in the start script.
The first deploy will apply the `20260324082751_init_schema` migration.

---

## Optional: Custom Domain
Railway → your service → **Settings** → **Domains** → Add custom domain.

---

## Notes on File Storage
- Without S3 configured, uploaded documents are stored locally inside the container.
- **Local storage is wiped on every redeploy.** For production use, configure AWS S3.
- See `.env.railway` in `/backend` for all S3 variable names.

