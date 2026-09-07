# Aurex International MUN — Web App

A full React (Vite) + Node/Express site for Aurex International MUN
(3–4 Oct 2026, Army Public School, Bhopal): a public landing page with a
real registration form and a UPI payment QR, plus a password-protected
Secretariat admin panel for tracking registrations and fee payments.

```
web/
  server/   Node + Express API (JSON-file datastore, no DB server needed)
  client/   React app (Vite) — public site + /admin panel
```

## Run it locally

**1. Backend**

```bash
cd web/server
npm install
cp .env.example .env
```

Open `.env` and set:
- `ADMIN_PASSWORD` — the Secretariat login password (**change this from the default**)
- `JWT_SECRET` — any long random string
- `PORT` — defaults to 4000

```bash
npm run dev
```

The API now runs at `http://localhost:4000`. Registration and payment-config
data is stored in `web/server/aurex-data.json` (created automatically on
first run) — back this file up before deploying a new version of the server.

**2. Frontend**

In a second terminal:

```bash
cd web/client
npm install
npm run dev
```

Open `http://localhost:5173`. The dev server proxies `/api/*` to the
backend on port 4000 (see `vite.config.js`), so both must be running.

## Using it

- **Public site** (`/`) — the landing page. The registration form at the
  bottom writes directly into the Secretariat's database.
- **Secretariat panel** (`/admin`, login at `/admin/login`) — sign in with
  `ADMIN_PASSWORD`. From here you can:
  - see every registration (public submissions **and** ones you log by hand),
    with live totals (schools, delegates, ₹ collected, pending payments)
  - add/edit/delete a registration, and record its fee status, amount paid,
    UTR and payment date
  - enter your real **UPI ID** once you have one — the panel generates a
    genuine scan-to-pay QR code (using the `qrcode` npm package) with the
    fee amount pre-filled, and it appears automatically on the public
    "Scan & Pay" section. Leave the UPI ID blank and the public page shows
    an explicit "placeholder, not yet active" graphic instead — nothing
    ever looks like a live payment method until you turn it on.

## Deploying

This is a normal two-service app — deploy it wherever you'd deploy any
Node + static React app:

1. **Backend**: any Node host (Render, Railway, Fly.io, a VPS, …).
   Set `ADMIN_PASSWORD`, `JWT_SECRET`, and `CLIENT_ORIGIN` (your deployed
   client's URL) as environment variables. `npm start` runs it.
   The JSON datastore is a single file — for anything beyond light use,
   swap `server/db.js` for a real database (the rest of the code only
   calls the functions it exports, so the swap is contained to that file).
2. **Frontend**: `npm run build` in `web/client` produces `dist/` — serve
   it as a static site (Vercel, Netlify, Cloudflare Pages, …), or point
   `CLIENT_ORIGIN`/your build's API base at the backend's URL. The Express
   server also serves `client/dist` directly if you deploy both together
   on one host (see the bottom of `server/index.js`).
3. Update `client/vite.config.js`'s proxy target (dev only) and, if you
   split hosts, add a `VITE_API_URL`-based base to `client/src/api.js`
   instead of the relative `/api` path used for same-origin deployment.

## Security notes before going live

- Change `ADMIN_PASSWORD` and `JWT_SECRET` in `.env` — the checked-in
  defaults are for local development only.
- The admin session cookie is `httpOnly` and marked `secure` automatically
  in production (`NODE_ENV=production`), so serve the deployed site over
  HTTPS.
- Registrations submitted from the public form are unauthenticated by
  design (any school can register) — the Secretariat panel is the only
  place that requires a password.
