# 2. Deployment: hosting, DNS, email, database

Last verified: 2026-11-05. Example stack: Render (API web service + static site), MongoDB Atlas (free tier), Namecheap DNS, Cloudflare in front (applied by the host), Resend, Cloudinary. Plans, limits and screen names change; confirm in each console.

## Topology that worked

- **One public hostname** (the apex domain). The static site rewrites `/api/*` to the API service, so the browser talks to one origin. `www` redirects to the apex.
- Rewrite order matters: `/api/*` and any dynamic file (sitemap) first, then one rule per real application route, and **no catch-all `/* → /index.html`** if you want real 404s (see phase 5).
- The API checks a same-origin guard against an explicit list of allowed origins.
- **A blueprint file is not the live configuration.** If the services were created by hand in the dashboard, the platform never reads `render.yaml`: rewrite rules and response headers must be copied into each service's settings, and they drift silently. Keep the file as the reference, and let the smoke script (phase 6) check the live result: missing security headers, a dynamic file served as the app, or a 200 for an unknown path all mean the dashboard and the file disagree.
- When you replace a catch-all rule with explicit routes, add the new rules **above** it first, verify, and delete the catch-all last, so no page breaks in between.

## Build and start commands

- If the platform sets `NODE_ENV=production` during build, `npm ci` skips dev dependencies and the TypeScript compiler may be one of them. Use `npm ci --include=dev` in the build command.
- Build shared packages first (`shared` → `game-core`/other libs → `api`), then the app.
- Health check path: a cheap endpoint that also reports the database state (`/api/health`). Use the same URL for an uptime monitor.
- Pin the Node major version through an environment variable.

## Environment variables (example set)

Server (secrets and URLs are set only in the host's dashboard; the repo is public):

- Runtime: `NODE_ENV=production`, `NODE_VERSION`.
- Database: connection string with an explicit database name (…`/dbname?retryWrites=true&w=majority`).
- Two **distinct** secrets ≥ 32 chars: one for session JWTs, one for signing long-lived links (for example QR codes). **Back up the long-lived one** in a password manager and offline; losing it invalidates everything signed with it.
- Public URLs: the canonical site URL and the allowed-origins list. Remove the platform's default `*.onrender.com` origin once the custom domain works.
- Mail: provider name, API key, from-address on a sending subdomain, contact recipient.
- Media: cloud name, API key, API secret.
- Proxy trust: hops count and the Cloudflare-header switch (see `03-real-ip-behind-proxy.md`).
- **Must not exist in production:** rate-limit multipliers, seed credentials, any "open the site to search engines" switch until launch day.

Static site (build-time variables; changing them needs a rebuild): public OAuth client ID, canonical site URL, indexing switch (unset until launch).

## DNS and domain

1. Apex `A` record to the host's IP; `www` `CNAME` to the host. Remove any registrar "URL redirect" record on the apex: it overrides your `A` record and causes certificate errors.
2. Add the custom domain in the host's dashboard and wait for the certificate (minutes).
3. Mail-sending provider records (SPF/DKIM TXT on a **sending subdomain**) and a DMARC TXT. Verify in the provider's dashboard before sending anything.
4. Inbound mail: a registrar forwarder alias (for example `contact@`) to a mailbox you read. Create the alias first, wait for propagation, then test; bounces with "user unknown" mean the alias does not exist yet.
5. Registrar hygiene: domain lock, auto-renew, WHOIS privacy, and 2FA on the registrar account **owned by the site owner**, with delegated access for the developer.

## Database (MongoDB Atlas free tier example)

- **No scheduled cloud backups on the free tier** (a private-preview add-on may exist; do not rely on it). Your backup is your own export script. Upgrading the plan can delete the cluster and its snapshots: export first.
- Create a least-privilege database user for the app, restrict network access as far as the host allows (free hosts have no fixed egress IP, so you may need to allow all addresses; compensate with a strong password and a dedicated user).
- Seed the first administrator from a trusted machine with an idempotent script that reads credentials from environment variables, never from files in the repo.
- **Windows DNS gotcha:** `mongodb+srv://` can fail in Node with `querySrv ECONNREFUSED` when the router's DNS refuses SRV lookups. Preload a tiny script through `NODE_OPTIONS=--require setdns.cjs` that calls `require('node:dns').setServers(['8.8.8.8','1.1.1.1'])`.
- Make sure the seed script targets the intended database: a shell that lacks the environment variable may silently fall back to a local development `.env`.

## Social sign-in (Google example)

- OAuth consent screen "External" starts in **Testing**: only listed test users can sign in. Publishing needs a Branding page with homepage, privacy policy and terms URLs on a domain you own; the homepage must describe the app (not only a login form) and link the **same** privacy policy URL as the consent screen; the policy must say how Google user data is used. Re-check Google's current requirements before applying.
- Authorized JavaScript origins must list every origin that renders the button.
- Loading the provider's script only after the user clicks the button avoids contacting the third party on page view (relevant to cookie notices).
- The provider's button renders in an iframe with its own colour scheme. On a dark page the browser may paint an opaque white box around it. Setting `color-scheme: light` on the button's container fixes it.

## Rollout order for a breaking API change

Deploy the API first and wait for `/health`, then the site. If the login response shape changed, the old API and new site cannot talk to each other during the gap; with no real users that gap is acceptable, otherwise version the endpoint.

## Free-plan realities

- Free web services sleep and may share a monthly hours quota; an uptime monitor keeps one awake but uses the quota. Do not spin up extra free services for rehearsals if production depends on the same quota. Static sites do not consume it.
- Move the API to a paid plan before real traffic.
- Rollback: redeploy the previous commit from the host's dashboard; practise once.
