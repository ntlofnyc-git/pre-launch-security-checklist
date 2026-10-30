# Pre-launch security report: <app name>

| | |
|---|---|
| **Date** | YYYY-MM-DD |
| **Reviewed by** | <name / tool> |
| **Version / commit** | <tag or short sha> |
| **Environment tested** | local / staging / production (read-only checks only) |

## Summary

| PASS | FAIL | N/A | UNKNOWN |
|---|---|---|---|
| 0 | 0 | 0 | 0 |

**Ready to launch:** YES / NO / YES after fixes <list>

## Scope

- **Stack:** <framework, language>
- **Hosting:** <provider, server, CDN>
- **Auth:** <method / provider>
- **Database:** <type, where it runs>
- **Payments:** <provider or none>
- **File uploads:** <yes / no, where stored>
- **Not reviewed:** <anything out of scope>

## Findings that need fixing (most severe first)

### [Critical / High / Medium / Low] #<n> <check name>: FAIL
- **Where:** `<file>:<line>` or URL
- **Problem:**
- **Risk:**
- **Fix:**
- **Test after fix:** <command or step + result>
- **Status:** open / fixed / accepted risk (who accepted, why)

<!-- copy the block above for each finding -->

## All 27 checks

| # | Group | Check | Result | Evidence / note |
|---|---|---|---|---|
| 1 | Logins | Proper authentication | | |
| 2 | Keys | Hide API keys | | |
| 3 | Data | Secure the database | | |
| 4 | Server | Enforce HTTPS | | |
| 5 | Logins | Hash passwords | | |
| 6 | Data | Validate forms on the server | | |
| 7 | Data | Protect against XSS | | |
| 8 | Server | Log and monitor suspicious activity | | |
| 9 | Logins | Lock down admin routes | | |
| 10 | Data | Parameterized queries | | |
| 11 | Keys | Check environment variables | | |
| 12 | Logins | Rate limiting | | |
| 13 | Server | Restrict CORS | | |
| 14 | Server | Security headers | | |
| 15 | Logins | Secure session cookies | | |
| 16 | Logins | CSRF protection | | |
| 17 | Logins | Password-reset links expire | | |
| 18 | Data | Restrict file upload types and sizes | | |
| 19 | Payments | Verify prices and payments | | |
| 20 | Server | Debug mode off in production | | |
| 21 | Code | Update vulnerable dependencies | | |
| 22 | Code | Remove unused packages | | |
| 23 | Server | Check for exposed files | | |
| 24 | Keys | Scan Git history for secrets | | |
| 25 | Keys | Revoke and replace compromised keys | | |
| 26 | Code | Thorough security review | | |
| 27 | Code | Test protections and write report | | |

## Needs a human

- [ ] <key rotation in provider dashboard>
- [ ] <host / DNS / firewall setting>
- [ ] <expert review for payments or personal data>

## Sign-off

| Role | Name | Date |
|---|---|---|
| Developer | | |
| Reviewer | | |
