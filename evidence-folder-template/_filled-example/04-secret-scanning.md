# Secret scanning — Acme Analytics

> Illustrative example for a fictional company. See the blank template at
> `../04-vulnerability-secrets/secret-scanning.md`.

Secret scanning **and push protection** are enabled org-wide on GitHub. Push protection blocks a
secret from reaching the repository in the first place; scanning catches anything historical.

## Evidence on file

| Artifact                                 | What it shows                                                   |
| ---------------------------------------- | --------------------------------------------------------------- |
| `secret-scanning-org-2026-06-18.json`    | Org `security_and_analysis`: scanning + push protection enabled |
| `secret-scanning-alerts-2026-06-18.json` | Open alerts (0 open) + the one resolved alert this quarter      |
| `remediation-2026-04-22.md`              | Write-up of the April incident: detected, rotated, closed       |

## Gathered from

```bash
gh api orgs/acme --jq '.security_and_analysis' > secret-scanning-org-2026-06-18.json
gh api repos/acme/app/secret-scanning/alerts \
  --jq '[.[] | {number, secret_type, state, resolution, created_at, resolved_at}]' \
  > secret-scanning-alerts-2026-06-18.json
```

## The remediation example (this is what buyers want)

On **2026-04-22**, push protection flagged a Stripe test key committed to a feature branch.
Timeline:

- 14:02Z — push protection blocked the push; developer notified inline.
- 14:09Z — key confirmed as a **test-mode** key (no production exposure); rotated in Stripe anyway.
- 14:20Z — branch history rewritten to drop the blob; alert resolved as `revoked`.

Net exposure to a default branch: none. Full write-up in `remediation-2026-04-22.md`.

## Buyer questions answered

> "Do you scan for secrets, and what happens when one is found?"

Yes — scanning + push protection org-wide. The most recent detection (2026-04-22) was a blocked push
of a test key; it was rotated and the history cleaned the same hour. See the remediation note.

## Refresh

Quarterly. Last refreshed 2026-06-18 by R. Mensah. Next due 2026-09-18.
