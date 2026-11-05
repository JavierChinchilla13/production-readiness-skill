# 8. Mistakes that cost time

Last verified: 2026-11-05. Real problems met during a first launch, with the fix. Generic details; no project data.

| Symptom | Cause | Fix |
|---|---|---|
| Build fails: `Cannot find type definition file for 'node'` | `NODE_ENV=production` made `npm ci` skip dev dependencies | `npm ci --include=dev` in the build command |
| Login returns 403 on the platform's default URL | Same-origin guard rejects origins not in the allow-list | Add that origin while testing, then remove it |
| Sign-in fails after setting a cookie domain | A parent-domain cookie does not apply on the platform's default hostname | Remove the cookie domain; host-only cookies work everywhere you serve |
| Certificate error on the apex domain | A leftover registrar "URL redirect" record on the apex overrode the `A` record | Delete the redirect record; keep `A` (apex) and `CNAME` (`www`) |
| Forwarded mail bounces "user unknown" | The forwarding alias was not created or had not propagated | Create the alias first, wait, then test |
| "Admin not found" after seeding | The seed ran against a local development database because the connection variable was unset in that terminal | Set every variable in the same terminal; verify the target database name before seeding |
| `querySrv ECONNREFUSED` on Atlas from Windows | Router DNS refuses SRV lookups | Preload `dns.setServers(['8.8.8.8','1.1.1.1'])` via `NODE_OPTIONS` |
| Rate limiter would block everyone | The app saw an internal proxy address for all visitors | Measure, then trust the CDN header (see phase 3) |
| Privacy text contradicted the logs | The HTTP logger wrote `X-Forwarded-For`, `CF-Connecting-IP` and the socket address on every line | Allow-list logged fields; test that IP headers never appear |
| Privacy text claimed a provider identifier was received | Written from a template, not from the code | Re-read the code; fix the text; generate text from code |
| Account takeover path | Linking social sign-in to an unverified password account | Track verified email; rotate password and revoke sessions on linking; re-ask declarations |
| Age brake blocked real users | Cap of 5 rejections/hour per network, but mobile carriers share one IP | Raise the cap (we used 15) and size limits for shared networks |
| Google button had white square corners | The iframe is light-scheme on a dark page, so the browser paints it opaque | `color-scheme: light` on the button container |
| "It sends me to admin" after Google sign-in | The tester's Google account was the admin's email | Test with another account; role comes from the database |
| Console showed 401s for anonymous visitors | `me` and `refresh` called on every visit | Session-hint cookie; skip calls when absent |
| Soft 404s | Catch-all rewrite to `index.html` | Explicit route rewrites + `404.html` + a drift test |
| Page had no `H1` when a list was empty | Heading lived inside the list component | Keep the page title outside conditional content; audit each route with a test |
| Hosting hours at risk | An extra free service for rehearsal shared the monthly quota | Rehearse locally or briefly; delete extra services |
| Gave instructions to turn off CDN features in an account that did not exist | Saw `Server: cloudflare` and assumed the project had its own CDN account; it was the host's CDN | Ask who owns the CDN before any CDN step; check the registrar's nameservers |
| Smoke test after deploy: no security headers, sitemap served as the app, unknown paths returned 200 | Services had been created by hand, so the blueprint's rules and headers were never applied | Copy rules and headers into the dashboard; keep the smoke test as the drift check |
| Asked another agent to build security fixes that already existed | The roadmap was out of date | Read the code (and its tests) before writing a task; update the roadmap when closing items |
| The owner's real name was about to be hard-coded in a public repo | Legal texts were filled straight into the code | Admin-only setting filled on the production site; fake names in tests |
| A claim about system state turned out wrong | Judged from a stale snapshot instead of checking | Check processes, timestamps or the live URL before asserting state; say "not verified" when you could not |

Habit that prevents most of these: **measure on the deployed stack, write the assumption down, and add a test that fails when it stops being true.**
