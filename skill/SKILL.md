---
name: pre-launch-security-check
description: Run the 27-point pre-launch security check on a website or app and produce a PASS / FAIL / N/A report with findings and fixes. Use when the user asks to "run the pre-launch check", "security check my app before launch", "is my app safe to go live", "run the 27 checks", or asks for a launch-readiness security review.
---

# Pre-launch security check (27 checks)

You are reviewing a website or app before it goes live. Work through all 27 checks below against the **real code and config in this project**, then write a report.

## Ground rules

1. **Read before you judge.** Find the actual code for each check (auth middleware, routes, DB queries, env files, server config, upload handlers, checkout). Cite files and line numbers.
2. **Every check gets a verdict:** `PASS`, `FAIL` or `N/A`. `N/A` needs a one-line reason (e.g. "no file uploads in this app"). Never skip a check silently. If you can't tell, say `UNKNOWN` and what you'd need to confirm it.
3. **Evidence, not vibes.** For PASS, say what you saw. For FAIL, say what's wrong, where, and why it matters.
4. **Fix only with permission.** Default to findings + proposed fixes. If the user asked you to fix, make one fix at a time, smallest change first, and re-test after each.
5. **Prove it.** Where you can safely run something locally (tests, `curl` against a local/dev server, an audit command, a secret scan), do it and include the output. Never attack a production system or third-party service, and never run destructive commands.
6. **Never print secrets.** If you find a key, show only the first 4 characters and the file/commit it's in, and tell the user to revoke and replace it (check #25).
7. **Humans sign off.** Remind the user that apps taking payments or storing personal data should also get a review from an experienced person.

## The checks

Order follows the numbers. Group in brackets.

| # | Check | Look for / how to test |
|---|---|---|
| 1 | Proper authentication [Logins] | Server verifies the session/token on every protected route; logged-out `curl` to a private endpoint gets 401/403. |
| 2 | Hide API keys [Keys] | No secret keys in client code, mobile bundles or built JS. Search for `sk_`, `secret`, `token`, provider key prefixes. |
| 3 | Secure the database [Data] | DB not exposed publicly; least-privilege DB user; row-level security on if the platform uses it. |
| 4 | Enforce HTTPS [Server] | HTTP redirects to HTTPS; HSTS; no mixed content. |
| 5 | Hash passwords [Logins] | bcrypt / scrypt / Argon2 with salt. No plain text, MD5 or SHA-1. |
| 6 | Validate forms on the server [Data] | Server-side schema/type/range checks on every input, not just browser checks. Ownership checks on IDs. |
| 7 | Protect against XSS [Data] | Output encoding; no raw HTML rendering of user input (`innerHTML`, `dangerouslySetInnerHTML`, `v-html`, `|safe`) without sanitizing. |
| 8 | Log and monitor suspicious activity [Server] | Failed logins, 403s and admin actions are logged and alerted; no passwords/tokens/cards in logs. |
| 9 | Lock down admin routes [Logins] | Role check on the server for every admin page and admin API, not just a hidden link. |
| 10 | Parameterized queries [Data] | No SQL built with string concatenation / template strings / f-strings around user input. |
| 11 | Check environment variables [Keys] | Nothing secret behind a public prefix (`NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`, `EXPO_PUBLIC_`...). `.env` files gitignored. |
| 12 | Rate limiting [Logins] | Limits on login, sign-up, reset, contact forms and costly API calls. 429 after the limit. |
| 13 | Restrict CORS [Server] | No `*` origin on cookie-auth or private endpoints; explicit allowlist. |
| 14 | Security headers [Server] | CSP, HSTS, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `frame-ancestors`/`X-Frame-Options`. |
| 15 | Secure session cookies [Logins] | `HttpOnly`, `Secure`, `SameSite`, sensible expiry, server-side logout. |
| 16 | CSRF protection [Logins] | Tokens or SameSite + origin checks on every state-changing request. |
| 17 | Password-reset links expire [Logins] | Random, single-use, short-lived tokens; same response for known and unknown emails. |
| 18 | Restrict file uploads [Data] | Type (by content, not just extension), size limit, stored outside web root, never executed, random names. |
| 19 | Verify prices and payments [Payments] | Server sets the price; payment confirmed via signed webhook or provider API before access is granted. |
| 20 | Debug mode off in production [Server] | No stack traces, debug toolbars or verbose errors in prod config. |
| 21 | Update vulnerable dependencies [Code] | Run the ecosystem audit (`npm audit`, `pip-audit`, `composer audit`, `cargo audit`, `govulncheck`...). High/critical fixed or justified. |
| 22 | Remove unused packages [Code] | Unused deps, demo routes, test pages and seed accounts removed. |
| 23 | Check for exposed files [Server] | `.env`, `.git/`, backups, SQL dumps, logs and unintended source maps not served by the web server. |
| 24 | Scan Git history for secrets [Keys] | `gitleaks detect --log-opts="--all"` or `trufflehog git file://.` over the full history. |
| 25 | Revoke and replace leaked keys [Keys] | Any key found in #2, #11 or #24 is revoked at the provider and replaced, not just deleted from code. |
| 26 | Thorough security review [Code] | List every route, endpoint, job and integration; confirm each is covered by the checks above. Look for gaps. |
| 27 | Test the protections and write a report [Code] | Re-test every fix and produce the report below. |

## How to run it

1. **Map the app first.** Stack, framework, hosting, auth method, database, payment provider, file storage, every route/endpoint. Write this as a short "Scope" section.
2. **Go group by group** (Logins, Keys, Data, Payments, Server, Code). For each check: find the code, decide the verdict, collect evidence.
3. **Run what's safe to run** locally: dependency audit (#21), secret scan (#24), unused-deps check (#22), `curl` against a local or staging server for #1, #4, #9, #12, #13, #14, #20, #23.
4. **Write the report** in the format below. Put FAILs first, ordered by severity (Critical, High, Medium, Low).
5. **Offer next steps:** the exact fixes, in order, and which ones need the user to act outside the code (revoking keys, changing host settings, turning on provider features).

## Report format

```markdown
# Pre-launch security report: <app name>
Date: <date> · Reviewed by: <you> · Commit: <short sha>

## Summary
PASS: <n> · FAIL: <n> · N/A: <n> · UNKNOWN: <n>
Ready to launch: YES / NO / YES after fixes 1-3

## Scope
<stack, hosting, auth, DB, payments, uploads, what was and wasn't reviewed>

## Findings that need fixing (most severe first)
### [Critical] #2 Hide API keys: FAIL
- Where: `src/lib/payments.ts:12`
- Problem: secret payment key is bundled into browser code.
- Risk: anyone can read it and charge or refund on your account.
- Fix: move the call to a server route; revoke the old key and issue a new one (#25).
- Test after fix: built JS bundle searched for `sk_`, no matches.

## All 27 checks
| # | Check | Result | Evidence / note |
|---|---|---|---|
| 1 | Proper authentication | PASS | `middleware.ts` verifies session on /api/*; logged-out curl → 401 |
| ... | ... | ... | ... |

## Needs a human
<anything outside the code: key rotation, provider dashboards, host settings, legal/compliance>
```

If the user asks you to fix things, update the report after each fix and re-test, so the final version shows what was found, what was changed and the proof it works.
