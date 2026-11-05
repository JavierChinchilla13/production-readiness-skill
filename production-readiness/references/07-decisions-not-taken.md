# 7. Decisions not taken, and when you would take them

Last verified: 2026-11-02. Each row is something we considered and deliberately did **not** do for a small, low-traffic, adult-and-teen readership with no payments. The right column says when your project should choose differently. Add your own decisions with `templates/adr-template.md`.

| Topic | What we did | Why | Choose the alternative when… |
|---|---|---|---|
| API on its own subdomain (`api.example.com`) | Same origin: the static site proxies `/api/*` | Fewer cookie/CORS problems; one hostname | The proxy does not deliver the real client IP, you serve a mobile app or third parties, or you need independent scaling and caching |
| Real age verification (ID, card, third-party service) | Declared month/year + checkboxes | Proportionate for a +16 book; minimal data | The content is adult-only, law demands verification, or there are payments/sensitive services |
| Analytics and cookie-consent manager | Strictly necessary cookies only, no analytics | Nothing to consent to; simpler | You add analytics, advertising or pixels (then you need prior consent where required, and a consent manager) |
| CAPTCHA | Per-network attempt limits | Friction hurts small communities | You see real automated abuse of sign-up or contact forms |
| Mandatory 2FA for users | Deferred; owner enables it on provider accounts | Small user base; staff accounts are few | Sensitive data, payments, many staff, or regulatory expectations |
| Enforcing a content-security-policy | Report-only | Avoid breaking third-party scripts blindly | Reports are clean for 1–2 weeks and rich-text/image policy is settled |
| Managed database backups (paid tier) | Own export script + manual rehearsal | Free tier has none; budget | Data cannot be lost, there is revenue, or more than one person depends on the data |
| Shared rate-limit store (Redis/TTL collection) | In-memory counters | One instance; limits are soft | Several instances, strict limits, or abuse that survives restarts |
| Email verification before account creation | Not required | Friction; accepted enumeration trade-off | Accounts hold something valuable, you must prevent registration with others' emails, or enumeration matters |
| Account lockout after failures | Not used | A stranger could lock out the owner | You pair it with unlock-by-email and per-IP delays, or a regulator expects it |
| Breached-password check | Not used | Extra dependency | Higher-value accounts; use a k-anonymity API |
| Server-side rendering / prerendering | Static SPA with per-page meta in the browser | Free static hosting | Link previews and SEO per page matter a lot (previews from crawlers that do not run JavaScript show only site-level data) |
| `llms.txt` and blocking AI crawlers | Left to the owner; template has a commented block | It is a content-owner choice | The owner wants to opt out of AI training or control citation |
| Prepaid/paid hosting from day one | Free tier + uptime monitor | Pre-launch, no real users | Before public launch; free tiers sleep and have hour quotas |
| Rehearsal environment (separate services) | Skipped | Consumed the shared free-hours quota; no real users | There are real users, a paid plan, or a risky migration |
| Secret header from the CDN to the API | Residual risk accepted | Only affects per-IP limits | You want to remove the "origin reachable without the CDN" assumption entirely |
| Reader-submitted reviews with moderation | Owner publishes reviews with permission | Simpler, lower legal exposure | You want community reviews; then design moderation criteria, policy, records and legal text first |
| Direct sales on the site | Link to third-party sellers | Avoids seller obligations | You sell yourself; get legal advice on seller identity, prices, returns and payments first |
| Video backgrounds, heavy animation libraries | CSS only, one narrow canvas exception | Performance and accessibility | Marketing-led sites where motion is the product, with reduced-motion and data-saver fallbacks |

Principle: write each skipped item down with its trigger. "We did not forget; we decided, and here is when to revisit."
