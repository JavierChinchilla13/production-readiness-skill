---
name: production-readiness
description: Use when taking a web application (especially a small full-stack app with user accounts) from "works locally" to a public launch. Walks through server-side security, deployment, real client IP behind proxies, age and consent flows, privacy/terms/cookies drafts, SEO and technical hygiene, backups and a launch checklist, and records the decisions that were deliberately NOT taken and when you would take them. Not legal advice; it tells you when to involve a lawyer.
---

# Production readiness

A field-tested checklist and decision log for launching a small web app. It was distilled from a real launch (React + Express + MongoDB on Render, with the host's Cloudflare in front, Resend for email, Cloudinary for media, Google sign-in). The stack is an example; the reasoning applies elsewhere.

**Read this first**

- This skill is **not legal advice**. Anything about privacy, minors, cookies, consumer law or reviews is a *draft to be reviewed by a qualified person in the right jurisdiction*. See `references/09-when-to-ask-a-professional.md`.
- Facts about third-party plans, limits and consoles change. Each reference file has a "Last verified" date. Re-check before relying on them.
- Never print, paste, commit or log secrets (connection strings, API keys, tokens, signing secrets, passwords). Use a password manager. If a command needs a secret, tell the human to set it themselves.

## How to use it

1. Ask the human what the jurisdiction and audience are (adults only? minors possible? payments? sensitive data?).
2. **Ask how they want to start**, before reading the code in depth. Show the phase table below and offer these options in one short question:
   - **(A) Full check:** you audit every phase and report what the app already has and what it lacks. This is the most complete option and the most expensive in time and tokens.
   - **(B) Choose phases:** the human picks the phases (or single items) to check, for example only security and backups. This is cheaper and focused.
   - **(C) Go straight to a task:** the human already knows what is missing and names it, for example "add the legal drafts". You still do a quick check of that one item first, so you don't duplicate or overwrite what already exists.
   Recommend one option with a one-line reason: (A) for a first launch, (B) when time or budget is tight. Wait for the answer. The scope it gives limits steps 3–4.
3. **Audit first, read-only.** Before proposing or changing anything, inspect the codebase, configuration and docs and check every item **in the chosen scope** against what the app **already has**. Translate each check to the app's own stack (a Spring Boot or Django app will not have Express middleware; look for the equivalent). Record each item as `Present`, `Partial`, `Missing` or `Not applicable`, always with evidence (`file:line`, a config key, a test name, or "searched X, Y, Z, found nothing"). Never mark an item `Present` from a name or a comment alone: open the code. Use `templates/audit-report.md`. Facts you cannot see from the code (cloud consoles, DNS, provider plans) are marked `Ask the human`.
4. **Stop and ask before implementing.** Show the audit report and ask the human, per phase, whether to (a) stop at the report, (b) fix only the `Missing`/`Partial` items marked high risk, or (c) implement everything missing in that phase. Say plainly that implementing takes noticeably more time and tokens than auditing, and give a rough size per phase (small / medium / large). Do not write code until the human answers; a "yes" for one phase does not cover the others.
5. Work **phase by phase** and stop at each review point. Do not push, deploy, publish, or change cloud consoles without the human's explicit order. Cloud and account steps are done by the human; you give exact, short instructions.
6. For every legal or policy text, use two visible markers:
   - `[PLACEHOLDER: …]` = a fact only the owner can provide (legal name, address, jurisdiction).
   - `[PROPOSAL: …]` = a value you propose with a technical reason, to be confirmed by a professional.
   Never invent owner facts. Never claim "compliant".
7. Prefer measuring to assuming (see real client IP). Prefer tests that tie configuration to code (route lists, env variables, policy hashes) so drift fails a test.
8. Close every phase with: what you did, what the human must do, and the next step, and say where each change can be seen.

## Phases

| # | Phase | Reference |
|---|-------|-----------|
| 1 | Architecture and security rules | `references/01-architecture-and-security.md` |
| 2 | Deployment (hosting, DNS, email, database) | `references/02-deploy-render-atlas.md` |
| 3 | Real client IP behind proxies and rate limits | `references/03-real-ip-behind-proxy.md` |
| 4 | Privacy, terms, cookies, age and consent | `references/04-legal-privacy-age-consent.md` |
| 5 | SEO and technical hygiene | `references/05-seo-and-technical.md` |
| 6 | Backups, smoke tests, launch checklist | `references/06-launch-checklist.md` |
| 7 | Decisions not taken, and when to take them | `references/07-decisions-not-taken.md` |
| 8 | Mistakes that cost time | `references/08-mistakes-we-made.md` |
| 9 | When to ask a lawyer or other professional | `references/09-when-to-ask-a-professional.md` |

Templates live in `templates/`: the audit report (step 3 above), an Architecture Decision Record, a Render blueprint example, a client-IP diagnostic endpoint, a smoke-test list, and the legal-marker conventions.

## Order that worked

1. Security rules and production start-up guards (phase 1) before anything is public.
2. Measure the real client IP on the deployed stack (phase 3) before trusting rate limits.
3. Draft legal texts and the age/consent flow (phase 4) in parallel with SEO (phase 5), keeping the site `noindex` until launch.
4. Backups and the smoke script (phase 6) **before** the first real data arrives.
5. Deploy, run the smoke script, then and only then open the site to search engines.

## Hard rules for the agent

- No code changes before the audit report is shown and the human has chosen what to implement (step 4).
- No push, force push, deploy, DNS change or account change without an explicit order in this conversation.
- No secrets in chat, files, logs, commit messages or screenshots.
- No claims of legal compliance, certification or "GDPR-ready". Use "draft pending legal review".
- No invented owner data, no invented reviews, no copyrighted assets.
- If a measurement contradicts an assumption, say so and change the plan.
