# 1. Architecture and security rules

Last verified: 2026-11-02. Example stack: React (Vite), Express, MongoDB, Zod. The rules are stack-independent.

## Rules that paid off

1. **The server is the source of truth** for permissions, progress and results. Hiding a button is not security. Every protected request re-checks session and entitlement on the server.
2. **Protected content never leaves the server before validation.** Do not ship it in the bundle, in a list endpoint or in a "locked" payload that happens to contain the text.
3. **Validate every input with a schema** (body, query, params) using shared schemas between client and server. No scattered manual validation.
4. **Layering**: routes → controllers → services → models. Controllers hold no business logic; services never see `req/res`; models are persistence only. Use atomic operations for state transitions (progress, counters).
5. **One-time secrets are stored only as hashes** (refresh tokens, password-reset tokens, signed links). Compare in constant time.
6. **Sanitize rich HTML on the server when saving**, and still treat it as untrusted when rendering.
7. **Errors have a stable shape** `{ error: { code, message } }`. Authentication errors are generic (do not reveal whether an email exists), and no internal details leak.
8. **Logs never contain** passwords, tokens, cookies, full emails, or client IPs if your privacy text says you do not keep them (see "Logging" below).
9. **Providers behind interfaces** (mail, object storage) so you can swap them. Free hosting tiers often block SMTP, so use an HTTP mail API.
10. **Roles are enforced by middleware**, not by routes being unlinked in the UI.

## Production start-up guards ("refuse to boot")

In production the process should exit with a clear message (never printing values) when:

- a required provider setting is missing (for example the media account name);
- two signing secrets are equal, too short, or contain a development placeholder;
- the mail provider is the in-memory fake;
- a testing-only multiplier or seed variable is set (rate-limit multipliers, seed credentials);
- public URLs are not `https` or point to localhost, or CORS contains wildcards.

Test each guard. A guard that is never exercised silently rots.

## Sessions and cookies

- Short-lived access token plus rotating refresh token, both in `HttpOnly; Secure; SameSite=Lax` cookies. Scope the paths narrowly (access on the API path, refresh only on the auth path).
- Same-origin deployment (the static site proxies `/api/*`) removes most CORS and cookie-domain problems. **Do not set a cookie domain** unless you really need subdomains; host-only cookies are safer and work on every hostname you serve.
- A same-origin guard that rejects unlisted `Origin` headers on state-changing requests is cheap defense in depth.
- Optional **session-hint cookie** (non-HttpOnly, no data, no security value) lets the browser skip `me`/`refresh` calls for anonymous visitors, so the console is free of 401 errors. Forging it must gain nothing; test that. Existing sessions will not have it after rollout, so users log in once again.

## Account linking and sign-in providers

- Linking a social login to an existing password account by verified email is convenient but opens **pre-hijacking**: someone registers a victim's email with a password they know; the victim later signs in with Google and lands in the attacker's account. Mitigations that worked: track `emailVerifiedAt`; when linking to an account whose email was never verified, **rotate the password, bump the token version and revoke refresh tokens**, and re-ask any declarations (age, consents) that someone else may have made. Do not apply it to staff accounts created by an administrator.
- Close the race where a password registration and a social sign-in happen at the same moment.
- A new user arriving through a social provider should not be created until they complete any required step (age, consents). Return a "needs profile" state and create nothing yet.

## Enumeration trade-off

A registration endpoint that answers "email exists" vs "created" leaks account existence. Login and password recovery can (and should) answer generically, but registration usually cannot without email confirmation first. Decide explicitly, write it down, and cap attempts per network. Do not set the cap so low that shared mobile networks (carrier-grade NAT) lock out many real people.

## Logging

HTTP loggers often write request headers by default. Headers such as `X-Forwarded-For`, `CF-Connecting-IP`, `User-Agent` and the socket address are personal data. Use an **allow-list** of fields to log, not a deny-list: unknown headers added by a proxy later will otherwise leak. Add a test that sends those headers and asserts they do not appear. Mask secrets that appear in URLs (for example tokens in a path) in error logs too.

## Content and uploads

- Signed direct uploads to the media provider with size and type caps; verify server-side what was uploaded.
- Never delete user media from the API on edit; use a manual, dry-run-by-default cleanup script.
- Per-user rate limits on expensive endpoints (media signing, inspections).

## Testing posture

- Every endpoint: happy path, validation failure, permission failure.
- Pure logic (scoring, progression, age calculation) gets unit tests.
- Browser tests only for key flows (register → unlock → play → result; admin → publish).
- Tie config to code with tests: route list vs hosting rewrite rules, environment variables vs the schema, content-security-policy hash vs built output, "no source maps in the production build".
