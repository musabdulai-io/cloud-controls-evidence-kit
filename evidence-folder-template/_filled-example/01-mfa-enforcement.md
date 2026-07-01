# MFA enforcement — Acme Analytics

> Illustrative example for a fictional company. See the blank template at
> `../01-access-controls/mfa-enforcement.md`.

All human access to Acme's identity provider (Google Workspace) requires 2-Step Verification,
enforced at the org level. Enforcement is policy-level, not opt-in.

## Evidence on file

| Artifact                           | What it shows                                             |
| ---------------------------------- | --------------------------------------------------------- |
| `mfa-policy-2026-06-12.json`       | Workspace 2SV enforcement policy export (Enforced = true) |
| `mfa-status-export-2026-06-12.csv` | Per-user 2SV status for all 47 accounts                   |
| `mfa-exceptions-2026-06-12.md`     | The two exception accounts + compensating controls        |

## Gathered from

Google Workspace Admin → Security → 2-Step Verification → Enforcement (policy export), and Reports →
Audit → Login filtered for `is_2sv = false`.

## Buyer questions answered

> "Are all employees MFA-enabled?"

Yes. 45 of 47 accounts enforce 2SV via hardware/phone. See `mfa-status-export-2026-06-12.csv`.

> "Are there exceptions?"

Two: one break-glass `super-admin` account and one CI service account. Both are documented in
`mfa-exceptions-2026-06-12.md`. Compensating controls: the break-glass credential is stored in a
sealed 1Password vault with access logging; the CI account uses a short-lived OIDC token, not a
password, and cannot log into the console.

## Refresh

Quarterly. Last refreshed 2026-06-12 by A. Okada. Next due 2026-09-12 (calendar event set).
