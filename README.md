# production-readiness-skill

A field-tested checklist and decision log for taking a small web app from "works on my machine" to a public launch, packaged as a skill for AI coding agents (Claude Code and compatible tools).

It was distilled from a real first launch (React + Express + MongoDB on Render, with the host's Cloudflare in front, Resend, Cloudinary, Google sign-in). The stack is an example; the reasoning transfers.

> **Not legal advice.** Anything about privacy, minors, cookies, consumer law or reviews is a draft to be reviewed by a qualified professional in the right jurisdiction. Facts about third-party plans and consoles change; every reference file shows a "Last verified" date.

## What is inside

```
production-readiness/
  SKILL.md                      entry point: when to use it, rules for the agent, phases
  references/
    01-architecture-and-security.md
    02-deploy-render-atlas.md
    03-real-ip-behind-proxy.md
    04-legal-privacy-age-consent.md
    05-seo-and-technical.md
    06-launch-checklist.md
    07-decisions-not-taken.md   what we skipped on purpose, and when you should not
    08-mistakes-we-made.md
    09-when-to-ask-a-professional.md
  templates/
    audit-report.md  adr-template.md  legal-markers.md  network-check.md  render.yaml.example
```

Highlights

- **You choose the scope:** at the start the agent asks whether you want a full check of everything, only the phases you pick, or one specific task.
- **Audit first, then ask:** within that scope, the agent first checks, read-only, what your app already has (`Present` / `Partial` / `Missing`, with file-and-line evidence) and only implements what you choose, phase by phase, because building costs far more time and tokens than checking.
- **Measure, don't assume:** how to find the real client IP behind proxies before trusting rate limits.
- **Privacy by construction:** age gate that stores nothing about people below the minimum, separate consents, logs that never contain IPs.
- **Honest drafts:** `[PLACEHOLDER]` (owner facts) and `[PROPOSAL]` (reviewed by a professional) markers instead of invented legal text.
- **Decisions not taken:** a table of things you may not need (CAPTCHA, 2FA, SSR, paid backups, …) with the trigger that means you do.
- **When to call a lawyer**, and how to prepare a cheap, useful consultation.

## Install

Copy the `production-readiness` folder into your skills directory, for example:

```bash
cp -r production-readiness ~/.claude/skills/
```

Then ask your agent to use the `production-readiness` skill when you prepare a launch.

## Using it well

1. Tell the agent your jurisdiction, audience (minors? payments? sensitive data?) and hosting.
2. Read the audit report before anything is built, and decide per phase: stop at the report, fix only the high-risk gaps, or implement everything missing.
3. Work phase by phase and stop at each review point.
4. Never let the agent push, deploy or change cloud consoles without your explicit order.
5. Keep secrets in a password manager. Do not paste them into chat.

## Español

Hay un resumen breve en [README.es.md](README.es.md).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Corrections of outdated facts and new "mistakes" or "decisions not taken" are especially welcome. Do not include secrets, personal data, client names or copyrighted material.

## License

MIT. See [LICENSE](LICENSE). The content is provided as is, without warranty and without legal, security or professional advice.
