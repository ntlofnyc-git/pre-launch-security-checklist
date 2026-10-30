# Generic prompt (any AI coding assistant)

Not using Claude Code? Paste this into Cursor, Copilot Chat, Windsurf, ChatGPT, Gemini or any assistant that can read your project. For best results, also attach or paste [CHECKLIST.md](../CHECKLIST.md).

---

```text
Before I launch this app, run a full pre-launch security check on it.

Work through these 27 checks against the real code and config in this project:

Logins and access: #1 proper server-side authentication, #5 hashed passwords
(bcrypt/scrypt/Argon2), #9 admin routes checked on the server, #12 rate limiting on
login/forms/costly APIs, #15 secure session cookies (HttpOnly, Secure, SameSite,
expiry), #16 CSRF protection, #17 password-reset links that expire and work once.

Keys and secrets: #2 no secret API keys in browser/mobile code, #11 nothing secret
behind a public env prefix, #24 scan the full Git history for leaked secrets,
#25 any leaked key revoked and replaced, not just deleted.

Data and forms: #3 database not public + least-privilege access, #6 server-side
validation of every input, #7 XSS protection, #10 parameterized queries only,
#18 file uploads restricted by type, size and content, never executed.

Payments: #19 server sets the price and confirms payment with the provider
(signed webhook) before granting access.

Server settings: #4 HTTPS enforced, #8 suspicious activity logged and alerted
(no secrets in logs), #13 CORS restricted to my own sites, #14 security headers
(CSP, HSTS, nosniff, Referrer-Policy, frame protection), #20 debug mode off in
production, #23 no .env, .git, backups or dumps downloadable.

Code and dependencies: #21 vulnerable dependencies updated, #22 unused packages
and demo code removed, #26 one full review for anything the checks missed,
#27 test every protection and write a report.

Rules:
- Give every check a verdict: PASS, FAIL, N/A (with reason) or UNKNOWN (say what
  you'd need). Don't skip any.
- Cite the files and lines you looked at. Evidence, not guesses.
- Never print a secret in full. Show the first 4 characters and where it is.
- Don't change code yet. List findings and proposed fixes, most severe first.
- Where it's safe to run locally (dependency audit, secret scan, curl against my
  local/dev server), run it and show the output. Never touch production.

Output a report with: Summary (counts + ready to launch yes/no), Scope,
Findings that need fixing (severity, where, problem, risk, fix, how to test the
fix), a table of all 27 checks, and a "Needs a human" list.
```

---

Then fix the findings **one at a time**, and after each fix ask the assistant to prove it works ("try the admin page logged out and show me the response"). Use [templates/report-template.md](../templates/report-template.md) for the final record.
