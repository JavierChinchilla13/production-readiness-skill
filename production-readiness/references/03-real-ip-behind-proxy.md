# 3. Real client IP behind proxies, and rate limits

Last verified: 2026-11-02. Measured on Render behind Cloudflare with a static-site rewrite in front of the API.

## Why it matters

If the server sees the proxy's address instead of the visitor's, a per-IP rate limiter counts **all visitors as one**. One abusive person (or just a busy launch day) then blocks everyone. The opposite mistake, trusting a client-supplied header, lets an attacker pick any IP, evade limits or lock out a victim's network.

## Do not guess: measure

1. Add an **admin-only diagnostic endpoint** (see `templates/network-check.md`) that returns, with every address **masked** (last octet hidden, never logged):
   - how many entries `X-Forwarded-For` has;
   - the address you would get for 0…5 trusted hops, and whether each looks internal (private ranges);
   - whether `CF-Connecting-IP` (or your CDN's equivalent) is present and a single valid IP;
   - which source the app is currently using.
2. Call it from the **deployed** site from two different networks (home wifi and mobile data). Different real IPs must appear.
3. Choose the configuration from the evidence.

What we saw: five entries in `X-Forwarded-For`; hops 1–2 were internal addresses that changed per request; hops 3–4 were CDN addresses; the CDN header carried the real client address, different per network. Counting hops alone would have rate-limited by an internal address.

## Configuration pattern

- `TRUST_PROXY_HOPS` (0–5) instead of a hard-coded `trust proxy 1`.
- `TRUST_CLOUDFLARE_HEADER=true|false` (default false): when true **and** the header is a single valid IP (no zone id, not repeated), use it as `req.ip`; otherwise fall back to the hops setting.
- Register the middleware immediately after `trust proxy` and **before** the logger and every limiter.
- Tests: simulate the proxy chain; invalid, repeated and spoofed headers; the fallback; the logger never prints the address.

## The assumption you must write down

Trusting the CDN header is safe only if **the origin is reachable only through the CDN**, because the CDN overwrites the header. On our host the platform's default URL also answered with `Server: cloudflare`, so spoofing it directly did not work; whether the origin can be reached by bypassing the CDN entirely could not be proven. We accepted the residual risk (it affects only per-IP limits) and documented it. To close it fully: a CDN rule adds a secret header and the API checks it before trusting the CDN header.

## Limits and shared networks

- Mobile carriers often put many users behind one address (carrier-grade NAT). A per-IP limit set for "one person" can block a whole carrier for an hour. Choose limits for the busiest realistic shared network, not for one person.
- In-memory limiters reset on every restart or free-plan suspension and are per instance. Use a shared store (Redis or a TTL collection) only if you need strict limits or run several instances.
- Authentication endpoints deserve their own tighter, but still shared-network-aware, limits; never lock an **account** after N failures if that lets a stranger lock out the owner (prefer per-IP plus delays).

## Re-check after changes

Re-run the diagnostic after any change to the CDN, the hosting plan, the rewrite rules or the domain setup.
