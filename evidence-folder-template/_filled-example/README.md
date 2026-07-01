# Filled example — what "done" looks like

The blank templates tell you _what_ to gather. This folder shows you what they look like once
they're **filled in**. Everything here is for a fictional company, **Acme Analytics** (a Series B
B2B SaaS on AWS + GCP + GitHub) — the names, dates, and numbers are illustrative, not real.

Use these as a reference for the level of concreteness auditors and buyers accept. Notice the
pattern in every file:

- **Specific artifacts named**, with dated filenames (`mfa-policy-2026-06-12.json`), not "we have a
  policy."
- **Real numbers** — row counts, wall-clock times, alert counts, dollar caps.
- **A named owner and a real last-refreshed date.**
- **Exceptions disclosed**, not hidden — auditors find them anyway.

| File                      | Control | What it demonstrates                            |
| ------------------------- | ------- | ----------------------------------------------- |
| `01-mfa-enforcement.md`   | 1.1     | IdP policy export + status report + exceptions  |
| `03-branch-protection.md` | 3.1     | Org ruleset export + CODEOWNERS                 |
| `04-secret-scanning.md`   | 4.3     | Scanning enabled + a real remediation example   |
| `05-restore-test.md`      | 5.2     | A completed quarterly restore test with metrics |
| `06-spend-caps.md`        | 6.3     | AI workload spend caps + alert routing          |

When you populate your own evidence folder, aim for this. If you'd rather an engineer produce a
folder like this from your actual stack, that's a
[Controls Review](https://musabdulai.com/sample-report).
