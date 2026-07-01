# 01 · Access controls

Evidence that the people and machines accessing production systems have appropriate, current
authentication and authorization.

## What goes here

- MFA enforcement evidence (IdP policy + per-user status)
- Quarterly IAM review minutes (who has admin, who shouldn't)
- Service-account / long-lived key inventory with rotation status
- Completed offboarding checklists (sample of 3-5 recent)

## Owner

Typically split: **IT / Security** owns the IdP layer; **Platform** owns the cloud IAM layer; **HR +
IT** jointly own offboarding.

## Common gotchas

- **Long-lived service-account keys.** Almost every audit catches at least one of these. Inventory +
  rotation policy + last-rotated date is the standard evidence. Migration to Workload Identity (GCP)
  or IAM Roles for Service Accounts (AWS EKS) is the long-term fix.
- **GitHub Personal Access Tokens with org admin.** Common blind spot. Audit org members' PATs and
  replace with GitHub Apps where possible.
- **MFA exceptions for "break-glass" accounts.** Document them, scope the credential storage, rotate
  quarterly. Don't pretend they don't exist.
- **Standing admin access for engineers.** Replace with just-in-time elevation via
  {{Teleport / AWS SSO / GCP IAM Conditions}} where possible. If not, document who and why.

## Cross-references

- Controls map: rows **1.1 – 1.5** in `../../controls-map.md`
- Platform guides: see `../../platforms/aws.md`, `gcp.md`, `azure.md`, `github.md`, `gitlab.md` for
  where to find each piece of evidence per platform
- Questionnaire answers: questions 1-3 in `../../questionnaire-answer-examples.md`
