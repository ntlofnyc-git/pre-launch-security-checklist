# The 27 pre-launch security checks

Run these on any website or app **before** it goes live. The numbers match the order in the video. They're grouped here by what they protect.

Each check has three parts:

- **What it is:** the protection in one line.
- **Why it matters:** what goes wrong without it.
- **How to test:** a quick way to prove it works. "Fixed" should mean you watched it work, not that someone changed some code.

Mark every check **PASS**, **FAIL** or **N/A** (with a reason) in the [report template](templates/report-template.md).

| Group | Checks | What it stops |
|---|---|---|
| [1. Logins and access](#1-logins-and-access-7-checks) | 7 | Account takeovers, open admin pages, password guessing |
| [2. Keys and secrets](#2-keys-and-secrets-4-checks) | 4 | Stolen API keys and surprise bills on your accounts |
| [3. Data and forms](#3-data-and-forms-5-checks) | 5 | Database leaks, injected code, malicious uploads |
| [4. Payments](#4-payments-1-check) | 1 | Tampered prices and fake orders |
| [5. Server settings](#5-server-settings-6-checks) | 6 | Snooping, leaked error details, downloadable backups |
| [6. Code and dependencies](#6-code-and-dependencies-4-checks) | 4 | Known holes in third-party code, untested fixes |

---

## 1. Logins and access (7 checks)

### #1 Set up proper authentication
- **What it is:** Sign-ins are checked by the server on every request, not just hidden behind a button in the browser.
- **Why it matters:** If the browser decides who is logged in, anyone can call your API directly and skip the login screen.
- **How to test:** Log out (or open a private window) and call a logged-in API endpoint with `curl`. You should get `401` or `403`, never data.

### #5 Hash passwords
- **What it is:** Passwords are stored as one-way hashes with a slow, salted algorithm (bcrypt, scrypt or Argon2), never as plain text or a fast hash like MD5/SHA-1.
- **Why it matters:** If the database leaks, plain or weakly hashed passwords hand attackers your customers' logins, often for other sites too.
- **How to test:** Create a test account and look at its row in the database. The password column should start with something like `$2b$`, `$argon2id$` or `scrypt`, and two users with the same password should have different hashes.

### #9 Lock down admin routes
- **What it is:** The server checks the user's role on every admin page and admin API call. Hiding the link isn't the same as locking the door.
- **Why it matters:** Bots try `/admin`, `/dashboard` and `/api/admin/*` on every site they find.
- **How to test:** Log in as a normal user and request each admin URL and admin API endpoint directly. Every one should return `403` (or redirect to login when logged out).

### #12 Add rate limiting
- **What it is:** Login, sign-up, password reset, contact forms and expensive API calls limit how many tries one visitor or IP gets.
- **Why it matters:** Without limits, bots guess passwords all day, flood your forms with spam and run up your email or AI bills.
- **How to test:** Send 20 wrong passwords in a row with a small script. After the limit you should see `429 Too Many Requests` or a lockout, and the limit should reset after the stated time.

### #15 Secure session cookies
- **What it is:** Login cookies are set with `HttpOnly`, `Secure` and `SameSite=Lax` (or `Strict`), and sessions expire on their own.
- **Why it matters:** A cookie readable by JavaScript or sent over plain HTTP can be stolen and used to take over the account.
- **How to test:** Log in, open the browser dev tools (Application > Cookies) and confirm the three flags. Check the expiry, and confirm logging out ends the session on the server, not just in the browser.

### #16 Add CSRF protection
- **What it is:** Actions that change things (email, password, payment method, deleting data) need a token that another site can't fake, or rely on `SameSite` cookies plus an origin check.
- **Why it matters:** Without it, a malicious page can make a logged-in visitor's browser submit a form to your app without them knowing.
- **How to test:** Replay a state-changing request without the CSRF token (or from a different origin). It should be rejected.

### #17 Make password-reset links expire
- **What it is:** Reset links are random, single-use and expire quickly (for example within an hour).
- **Why it matters:** An old reset link sitting in an inbox or a log is a permanent key to the account.
- **How to test:** Request a reset, use the link once, then try it again. Then request another and wait past the expiry. Both should fail. Requesting a reset for an unknown email should show the same message as for a known one.

---

## 2. Keys and secrets (4 checks)

### #2 Hide API keys
- **What it is:** Secret keys for payments, email, AI and other services live on the server and never ship in browser code or mobile app bundles.
- **Why it matters:** Anything in the browser can be read by anyone. Leaked keys get used within hours, and you get the bill.
- **How to test:** Load the site, open dev tools > Sources (or download the JS bundle) and search for `sk_`, `key`, `secret`, `token` and your providers' key prefixes. Only publishable keys meant for the browser should appear.

### #11 Check environment variables
- **What it is:** Anything with a "public" prefix (`NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`, `EXPO_PUBLIC_`...) really is safe to be public. Secrets are server-side only.
- **Why it matters:** Frameworks bake public-prefixed variables straight into the browser bundle. One wrong prefix leaks the key.
- **How to test:** List every environment variable and its prefix. For each public one, ask "would I post this on social media?" If not, move it server-side and rotate it (see #25).

### #24 Scan the Git history for leaked secrets
- **What it is:** Scan every commit, not just today's code, for keys, passwords and tokens.
- **Why it matters:** A key deleted from the code still sits in an old commit, and anyone with the repo can find it.
- **How to test:** Run a secret scanner over the full history, for example `gitleaks detect --source . --log-opts="--all"` or `trufflehog git file://. --only-verified`, and turn on GitHub secret scanning for the repo.

### #25 Revoke and replace compromised keys
- **What it is:** If a key ever leaked (in code, a commit, a log, a screenshot or a chat), cancel it at the provider and issue a new one.
- **Why it matters:** Deleting a leaked key from the code doesn't stop anyone who already copied it.
- **How to test:** For every key found in #2, #11 or #24, confirm in the provider's dashboard that the old key is revoked and the app works with the new one.

---

## 3. Data and forms (5 checks)

### #3 Secure the database
- **What it is:** The database isn't reachable from the internet, the app connects with the fewest permissions it needs, and row-level rules (if your platform uses them) are switched on.
- **Why it matters:** An open database port or an over-privileged connection turns one bug into a full data leak.
- **How to test:** From outside your network, try to connect to the database port (`nc -vz your-host 5432` / `3306`). It should fail. If you use a hosted backend with row-level security, query another user's rows with a normal user's token. You should get nothing back.

### #6 Validate forms on the server
- **What it is:** The server re-checks every value: type, length, range and format. Browser checks are only for convenience.
- **Why it matters:** Anyone can skip browser validation and send "-3 seats", a 10 MB name or an order for someone else's account.
- **How to test:** Send requests straight to the API with bad values (negative numbers, huge strings, wrong types, other users' IDs). Each should be rejected with a clear error, not saved.

### #7 Protect against XSS
- **What it is:** Anything a visitor types (names, comments, messages) is shown as text and can never run as code on someone else's screen.
- **Why it matters:** Injected script can steal sessions, change pages or act as the logged-in user.
- **How to test:** Put `<img src=x onerror=alert(1)>` and `<script>alert(1)</script>` into every text field that is displayed somewhere, including admin views. You should see the text, never a pop-up. Look for raw HTML rendering such as `dangerouslySetInnerHTML`, `v-html` or `innerHTML`.

### #10 Use parameterized queries
- **What it is:** User input is passed to the database as data with placeholders, never pasted into the query string.
- **Why it matters:** String-built queries allow SQL injection, which can dump or delete the whole database.
- **How to test:** Search the code for queries built with `+`, template strings or f-strings around user input. Then put `' OR '1'='1` into search and login fields. Nothing unusual should happen.

### #18 Restrict file upload types and sizes
- **What it is:** Uploads are checked for allowed type, size and real content, stored outside the web root (or in object storage) and never executed.
- **Why it matters:** A "photo.jpg" that is really a script, or a 5 GB upload, can take over or take down the server.
- **How to test:** Upload a `.php` / `.html` / `.svg` renamed to `.jpg`, a file over the size limit and a file with a double extension (`shell.php.jpg`). All should be refused, and uploaded files should download rather than run.

---

## 4. Payments (1 check)

### #19 Verify prices and payments
- **What it is:** The server sets the price from its own records and confirms each payment with the provider (signed webhook or API lookup) before granting anything.
- **Why it matters:** If the browser sends the price, a shopper can edit the page and pay $1.00 for a $149 plan.
- **How to test:** Intercept the checkout request and change the price, quantity or plan ID. The charge should still match the real price. Then send a fake "payment succeeded" webhook without a valid signature. It should be rejected.

---

## 5. Server settings (6 checks)

### #4 Enforce HTTPS
- **What it is:** Every page and API call loads over an encrypted connection, and plain HTTP redirects to HTTPS.
- **Why it matters:** Plain HTTP lets anyone on the same Wi-Fi read or change passwords, cookies and data.
- **How to test:** `curl -I http://your-site` should return a `301`/`308` to `https://`. Check the certificate is valid and not close to expiry, and that no page loads mixed (HTTP) content.

### #8 Log and monitor suspicious activity
- **What it is:** Failed logins, permission errors, admin actions and unusual traffic are logged, and someone gets an alert. Logs never contain passwords, tokens or card numbers.
- **Why it matters:** You can't respond to an attack you can't see.
- **How to test:** Trigger 10 failed logins and one `403` on an admin route, then confirm they appear in the logs and the alert fires. Search the logs for a test password to make sure it isn't recorded.

### #13 Restrict CORS settings
- **What it is:** Only your own approved sites can call your API from a browser. No `Access-Control-Allow-Origin: *` on anything that uses cookies or returns private data.
- **Why it matters:** A loose CORS policy lets any website read your API responses using your visitors' logins.
- **How to test:** `curl -H "Origin: https://evil.example" -I https://your-api/endpoint` should not echo that origin back in `Access-Control-Allow-Origin`.

### #14 Add security headers
- **What it is:** Send `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Referrer-Policy` and frame protection (`frame-ancestors` or `X-Frame-Options`).
- **Why it matters:** These headers tell browsers what's allowed to load and how, which blocks whole classes of attacks.
- **How to test:** `curl -I https://your-site` and check each header, or use a free header scanner. Confirm the site still works with the policy on.

### #20 Disable debug mode in production
- **What it is:** Visitors see a friendly error page, not a stack trace, file paths, framework versions or database details.
- **Why it matters:** Debug output is a map of your system for attackers.
- **How to test:** Visit a page that doesn't exist and force an error (bad ID, malformed JSON to the API). You should get a plain error with no internals. Check the production config has debug off.

### #23 Check for exposed files
- **What it is:** Backups, `.env` files, database dumps, `.git` folders, logs and source maps you didn't mean to publish can't be downloaded from the website.
- **Why it matters:** Bots request `/.env`, `/.git/config` and `/backup.zip` on every site, all day.
- **How to test:** Request `/.env`, `/.git/config`, `/backup.zip`, `/db.sql`, `/config.php.bak` and `/*.map` paths on the live site. All should return `404` or `403`.

---

## 6. Code and dependencies (4 checks)

### #21 Update vulnerable dependencies
- **What it is:** Third-party packages with known security holes are updated or replaced before launch.
- **Why it matters:** Known holes come with ready-made attack tools. Old packages are one of the most common ways in.
- **How to test:** Run your ecosystem's audit (`npm audit`, `pnpm audit`, `pip-audit`, `composer audit`, `bundle audit`, `cargo audit`, `govulncheck`) and fix every high and critical finding, or write down why it doesn't apply.

### #22 Remove unused packages
- **What it is:** Delete dependencies, routes, test pages, sample code and accounts the app no longer uses.
- **Why it matters:** Less code means fewer places for a problem to hide and fewer packages to keep patched.
- **How to test:** Run an unused-dependency tool (for example `npx depcheck` or `npx knip`), remove what isn't needed and confirm the app still builds and passes its tests. Check for leftover demo users and test endpoints.

### #26 Run a thorough security review
- **What it is:** One full pass over the whole system: sign-in, secrets, data, network, payments, files, inputs and packages, looking for anything the checks above missed.
- **Why it matters:** Individual checks catch known patterns. A review catches the gaps between them, like a forgotten endpoint or a new feature that skipped the rules.
- **How to test:** List every route, endpoint, background job and third-party integration, and confirm each one is covered by the checks above. If you use an AI coding assistant, ask it for a full security review with findings, then check them yourself.

### #27 Test the protections and write a report
- **What it is:** Every fix is tested to prove it works, and the results go into a short report of findings and fixes.
- **Why it matters:** "Fixed" should mean proven, not assumed. The report is your record for clients, auditors and the next launch.
- **How to test:** Re-run the "how to test" step for every check, record PASS / FAIL / N/A with evidence in [templates/report-template.md](templates/report-template.md), and keep the report with the release.

---

## Going further

These 27 checks cover the common launch-day mistakes. For a deeper standard, see the [OWASP Top 10](https://owasp.org/www-project-top-ten/), the [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) and CISA's [Secure by Design](https://www.cisa.gov/securebydesign) guidance. For anything that takes payments or stores personal data, have an experienced person review the result.
