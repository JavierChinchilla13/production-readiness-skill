# 6. Backups, smoke tests and launch checklist

Last verified: 2026-11-05.

## Backups (do this before real data arrives)

- A script that **reads only** and exports each collection/table to compressed files plus a **manifest** (counts, SHA-256 per file, indexes). It writes outside the repository, in a git-ignored location, never prints the connection string, and excludes live sessions and reset links unless asked.
- A restore script that **verifies hashes and counts before touching anything** and refuses any target that is not clearly a local test database (name must contain `test`, `local` or similar; refuse the production host and name). It never merges into existing data.
- Rehearse the restore into a local test database, then start the app against it and sign in. A backup you never restored is a hope.
- Backups hold personal data (emails, birth month/year, password hashes). Store them privately; encrypt before putting them in any cloud.
- Code backup: `git bundle` after each phase, verified, copied to an external disk or private storage. Code and data are different backups; you need both.
- Back up the **long-lived signing secret** separately (password manager plus offline copy).

## Smoke test script (read-only)

`smoke:prod <url>` should check, with no credentials: health (database up), a public list endpoint, a fake login returning 401 (not 403 or 500), security headers, `www → apex` redirect, `robots.txt` in the expected mode, every app route returns the app, an unknown path returns a real 404, anonymous visits receive no cookies, and the rate-limit counter restarts when called from another network. Exit code 1 on any failure. It writes no data (the fake login is the only non-GET).

## Pre-launch content check

A script that reads the database and reports sample or placeholder content visible to the public (placeholder tags, "Book 1", example emails, lorem text, demo reviews and fan art). It masks emails in its output.

## Checklist

**Security**
- [ ] Production start-up guards pass; no testing variables set.
- [ ] Distinct strong secrets; long-lived secret backed up.
- [ ] Same-origin allow-list correct; platform default origin removed.
- [ ] Logs verified to contain no IPs, tokens, cookies or full emails.
- [ ] Admin accounts: strong passwords, 2FA on provider accounts (registrar, host, mail, cloud console, source control).

**Infrastructure**
- [ ] Custom domain, certificate valid, `www` redirect, HSTS.
- [ ] Find out **who controls the CDN**. If it is your own account: SSL strict, and auto-minify and script rewriting off if you use a script-hash policy. If the host puts its own CDN in front (the response says `Server: cloudflare` but you have no account), there is nothing for you to configure there: do not give CDN instructions.
- [ ] Static-site rules and headers in the dashboard match the reference file (the smoke script checks headers, the dynamic sitemap and a real 404).
- [ ] Uptime monitor on the public health URL.
- [ ] Paid plan before real traffic (free tiers sleep and have quotas).
- [ ] Rollback practised once.
- [ ] Mail: sending subdomain verified, test message received, inbound alias works.

**Data**
- [ ] Backup and restore rehearsed; schedule decided (who runs it, how often, where it is stored).
- [ ] Backup retention matches what the privacy text promises; old copies are pruned automatically (and copies kept offline are pruned by hand).
- [ ] Sample content removed; pre-launch check clean.
- [ ] Deletion works end to end.

**Legal and content**
- [ ] Owner named; all `[PLACEHOLDER]` filled; `[PROPOSAL]` reviewed by a professional.
- [ ] Texts match the code and provider list.
- [ ] Age/consent flow tested for each band and for social sign-in.
- [ ] Cookie table complete; no tracking added without consent.
- [ ] Image and font licences confirmed.

**Search**
- [ ] Per-page metadata, canonical, sitemap, structured data verified.
- [ ] Indexing switches still OFF until the owner says go; then ON, redeploy, smoke with the indexable option.

**Launch day**
- [ ] Fresh backup. [ ] Deploy API, wait for health, deploy site. [ ] Smoke. [ ] Manual test of sign-up, sign-in, social sign-in, password reset, a 404 and the legal pages in a private window. [ ] Open indexing. [ ] Watch logs and CSP reports.
