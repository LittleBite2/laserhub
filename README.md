# Laser Hub website

Laser Hub is a React + Vite storefront. This repository now includes a separate Express + MongoDB Atlas API for production authentication.

## Project folders

- `src/` — Vite storefront, authentication pages and API client.
- `server/` — Express API, MongoDB user model, server-side sessions, password reset email and optional Google OAuth.
- `src/services/mockAuth.js` — local demo auth only; never used by a production build.
- `src/services/apiAuth.js` — browser client for the real API.

## Run the frontend

```bash
npm install
cp .env.example .env
npm run dev
```

On Windows PowerShell, use `Copy-Item .env.example .env` instead of `cp`. The example `.env` sets `VITE_AUTH_MODE=api`, so start the backend too. To use demo mode locally, change `VITE_AUTH_MODE=mock`; mock mode is only available in the Vite development server. Production builds always use the API.

## Run the API

```bash
cd server
npm install
cp .env.example .env
npm run dev
```

On Windows PowerShell, use `Copy-Item .env.example .env`. Fill in the server `.env` values from `server/.env.example` before starting. The API listens on port 5000 by default. Check `http://localhost:5000/api/health`.

### Required server variables

- `MONGODB_URI` — MongoDB Atlas connection string. Use a dedicated database user with least privilege and URL-encode special characters in the password.
- `SESSION_SECRET` — unique random secret of at least 32 characters.
- `FRONTEND_URL` — comma-separated allowed frontend origins, with no trailing slash.
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `EMAIL_FROM` — required to send real password-reset links.
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_CALLBACK_URL` — optional; configure the callback URL in Google Cloud Console to enable Google sign-in.

Do not commit `.env` files or credentials. Use host environment variables in deployment rather than uploading secrets.

## Authentication included

- Registration and login with bcrypt password hashes.
- Server-side sessions stored in MongoDB and delivered via HttpOnly cookies.
- Logout, current-session lookup, profile update and password change. Changing a password invalidates other active sessions.
- One-time, hashed, 30-minute password-reset tokens emailed through SMTP.
- Optional Google OAuth using the authorization-code flow and verified Google identity tokens.
- Login, registration and password-reset rate limits; origin allow-list and security headers.

### Production cookie / domain note

The production API uses `Secure` and `HttpOnly` cookies. The frontend and API must both use HTTPS. For Vercel/Netlify frontend plus a separately hosted API, browsers may restrict cross-site cookies when the providers use different base domains. For the most reliable deployment, use a custom domain with the frontend and API on the same site (for example, `www.example.com` and `api.example.com`), configure the exact frontend origin in `FRONTEND_URL`, and test sign-in in the browsers your customers use.

## Suggested deployment

- Frontend: Vercel or Netlify, root directory `.`; build command `npm run build`; output directory `dist`.
- Backend: deploy `server/` as a Node web service (for example, Render); build command `npm install`; start command `npm start`; health check `/api/health`.
- Database: MongoDB Atlas. Add only the backend host to Atlas network access when supported; avoid unrestricted network access if your hosting setup permits narrower rules.

Set the frontend's `VITE_API_URL` to the public HTTPS API origin. Set the backend's `FRONTEND_URL` to the deployed frontend origin. Environment variables prefixed with `VITE_` are public and must never contain passwords, API secrets or MongoDB credentials.

## SPA routing

`vercel.json` and `public/_redirects` are included for React Router history fallback on Vercel and Netlify.

## Before accepting real orders

- The storefront's orders, quotes, wishlist and cart are not yet backed by an authenticated server-side commerce database.
- `src/services/payment.js` does not process live payments. Add a server-side payment gateway integration and order verification before accepting payments.
- Update `src/config.js` with real business contact details. Confirm all product prices, sizes, materials, stock, lead times, shipping, returns and legal copy before launch.
