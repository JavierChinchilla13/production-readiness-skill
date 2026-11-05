# Production-readiness audit report (template)

Fill this in **read-only** before changing anything (SKILL.md, step 3), covering only the scope the human chose in step 2 (full check, chosen phases, or one task). One row per check. Evidence is mandatory: a reviewer must be able to open it and agree.

**App:** … · **Stack:** … · **Audited at commit:** … · **Date:** YYYY-MM-DD
**Scope chosen:** (A) full check / (B) phases … / (C) task … · **Jurisdiction / audience:** … (minors? payments? sensitive data?)

## Status legend

| Status | Meaning |
|---|---|
| `Present` | Implemented and you opened the code (or test) that proves it. |
| `Partial` | Exists but incomplete, untested, or only on some routes/environments. Say what is missing. |
| `Missing` | Searched and not found. List what you searched. |
| `Not applicable` | Does not apply to this app. Say why in one line. |
| `Ask the human` | Lives outside the code (cloud console, DNS, provider plan, legal fact). |

## Phase N: name

| # | Check | Status | Evidence | Risk if missing | Size to fix |
|---|---|---|---|---|---|
| N.1 | Example: refresh tokens stored only as a hash | Present | `auth/TokenService.java:88`, test `TokenServiceTest#storesHash` | High | — |
| N.2 | Example: rate limit on password reset | Missing | searched `reset`, `RateLimit`, `Bucket4j`, `limiter`: nothing on `/auth/forgot` | High | Small |
| N.3 | Example: backups tested by a restore | Ask the human | no backup script in repo; provider plan unknown | High | Medium |

(Repeat per phase.)

## Summary

- **Highest risks first:** the 3–5 `Missing`/`Partial` items that matter most, one line each.
- **Already solid:** what the app does well (so nobody rewrites it).
- **Questions for the human:** everything marked `Ask the human`.

## Decision for the human

For each phase, choose one:

- **(a)** Stop at this report.
- **(b)** Fix only the high-risk `Missing`/`Partial` items.
- **(c)** Implement everything missing in this phase.

Implementing costs noticeably more time and tokens than auditing. Sizes above are rough: small ≈ under an hour, medium ≈ a few hours, large ≈ a day or more.
