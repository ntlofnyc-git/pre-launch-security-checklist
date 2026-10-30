# Security

This repo is a checklist, a report template and an AI assistant skill. It contains no code that
runs on your servers.

- **Test only what you own.** Run the "how to test" steps against your own local, staging or
  production systems, never against sites or services you don't have permission to test.
- **Don't paste secrets into an AI chat.** If the skill finds a key, it shows only the first
  few characters. Revoke and replace any key that was ever exposed (check #25).
- **A checklist is a first pass, not a guarantee.** For apps that take payments or store
  personal data, also get a review from an experienced person.

Found a mistake in a check (a test that doesn't prove what it claims, or advice that's out of
date)? Please open an issue. If it's sensitive, open a private security advisory on this repo
instead.
