# GitHub evidence guide

Per-control reference for gathering compliance evidence in GitHub. Most engineering-led SOC 2
readiness work has GitHub as the critical-control surface for code review, deploy approvals, and
secret scanning.

## 1. Access controls

### Org membership + MFA (control 1.1)

**UI**: Org → People → Outside collaborators tab. Org → Settings → Authentication security →
"Require two-factor authentication".

**CLI**:

```bash
# Org-level 2FA enforcement
gh api orgs/<org> --jq '.two_factor_requirement_enabled'

# Members with 2FA disabled (requires admin scope)
gh api "orgs/<org>/members?filter=2fa_disabled" --jq '.[].login'

# Org owners — use the documented `role` query param. The /members list
# response is a Simple User object with NO `role` field, so filtering it
# with `select(.role=="admin")` always returns nothing. Pass ?role=admin.
gh api "orgs/<org>/members?role=admin" --jq '.[].login'
```

**Evidence**: confirm `two_factor_requirement_enabled = true` org-wide. This is the right control;
it prevents anyone from being a member without 2FA.

### Personal Access Tokens

PATs are a common blind spot. Even with 2FA, a leaked PAT lets an attacker bypass it.

**Read this before you file PAT evidence — coverage is not what most people assume:**

- **Fine-grained PATs** targeting your org emit `personal_access_token.*` audit events (access
  requested/granted/revoked). These you can evidence from the audit log.
- **Classic PATs** are user-owned credentials that are **not** created inside your org, so their
  creation does not appear in your org audit log. You can see classic PATs _act_ on the org (the
  resulting events), but you cannot enumerate "every classic PAT with access" from the audit log
  alone. An empty query result is **not** evidence that no classic PATs exist.
- Audit-log API access requires an org on a plan that exposes it (Enterprise Cloud), and a token
  with `read:audit_log`.

```bash
# Fine-grained PAT lifecycle events (org audit log; needs read:audit_log)
gh api "orgs/<org>/audit-log?phrase=action:personal_access_token" --paginate \
  --jq '.[] | {created_at, action, actor, token_id, programmatic_access_type}'

# The org's fine-grained PAT policy + pending/active grants (the authoritative
# list for fine-grained tokens — not the audit log)
gh api orgs/<org>/personal-access-tokens --paginate \
  --jq '.[] | {owner: .owner.login, permissions: .permissions, access_granted_at}'

# OAuth apps + GitHub Apps installed on the org
gh api orgs/<org>/installations --paginate --jq '.installations[].account.login'
```

**Reference**:
[Audit log events for your organization](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/audit-log-events-for-your-organization)
— confirm the current action names there; GitHub renames and adds events over time.

**Best practice**: restrict or block classic PATs org-wide (Settings → Personal access tokens →
restrict access via fine-grained tokens / deny classic), which turns "we can't enumerate classic
PATs" into "classic PATs can't reach us" — a control you _can_ evidence. Then replace PATs with
GitHub Apps (org-installable, auditable, fine-grained permissions). Document any remaining PATs with
owner + rotation date.

### CODEOWNERS (control 3.1 dependency)

```bash
# Pull the canonical CODEOWNERS
gh api repos/<owner>/<repo>/contents/.github/CODEOWNERS \
  --jq '.content' | base64 -d
```

## 3. Change management

### Branch protection / Rulesets (control 3.1)

GitHub has two systems: legacy "Branch Protection" and newer "Rulesets" (more flexible,
org-applicable). Rulesets are the direction; both are auditor-acceptable.

```bash
# Legacy branch protection
gh api repos/<owner>/<repo>/branches/main/protection > branch-protection-main.json

# Newer rulesets
gh api repos/<owner>/<repo>/rulesets > rulesets.json
gh api repos/<owner>/<repo>/rulesets/<id> > ruleset-detail.json

# Org-level rulesets (apply to all repos in the org)
gh api orgs/<org>/rulesets > org-rulesets.json
```

**Org-level rulesets** are the strongest evidence — they apply the same protection across every
repository, so adding a new repo doesn't accidentally create an unprotected production branch.

### Environment protection (control 3.2)

```bash
# Environments on a repo
gh api repos/<owner>/<repo>/environments > environments.json

# Production env detail (required reviewers, deployment branch
# restrictions, secrets scope)
gh api repos/<owner>/<repo>/environments/production > env-production.json
gh api repos/<owner>/<repo>/environments/production/deployment-protection-rules > prod-protection-rules.json
```

### Deploy history (control 3.4)

```bash
# Last N workflow runs on main
gh run list --workflow=deploy.yml --branch=main --limit=30 \
  --json databaseId,displayTitle,headSha,createdAt,actor,conclusion \
  > recent-deploys.json

# Per-deploy approver
gh api repos/<owner>/<repo>/actions/runs/<id>/approvals
```

### Signed commits (bonus credibility)

```bash
# Branch protection rule: require signed commits
gh api repos/<owner>/<repo>/branches/main/protection \
  --jq '.required_signatures.enabled'

# Verify a specific commit
gh api repos/<owner>/<repo>/commits/<sha> --jq '.commit.verification'
```

## 4. Vulnerability + secret scanning

### Dependabot (control 4.1)

```bash
# Dependabot config in repo
gh api repos/<owner>/<repo>/contents/.github/dependabot.yml \
  --jq '.content' | base64 -d

# Dependabot alerts
gh api repos/<owner>/<repo>/dependabot/alerts \
  --jq '[.[] | {number, severity, state, package: .dependency.package.name, fix_resolved: .fixed_at, created_at}]' \
  > dependabot-alerts.json
```

### Secret scanning + push protection (control 4.3)

```bash
# Org-wide secret scanning + push protection
gh api orgs/<org> --jq '.security_and_analysis'

# Per-repo
gh api repos/<owner>/<repo> --jq '.security_and_analysis'

# Open alerts
gh api repos/<owner>/<repo>/secret-scanning/alerts \
  --jq '[.[] | {number, secret_type, state, resolution, created_at}]'
```

**The control that matters**: `secret_scanning_push_protection` set to `enabled` org-wide. Push
protection prevents the secret from reaching the repo in the first place.

### Code scanning (CodeQL / SAST)

```bash
# Enable
gh api repos/<owner>/<repo>/code-scanning/default-setup -X PUT \
  --field state=configured

# Alerts
gh api repos/<owner>/<repo>/code-scanning/alerts \
  --jq '[.[] | {number, rule_id, severity, state}]'
```

## Audit log

GitHub Enterprise / GHE Cloud has an org-level audit log API. Pull this quarterly for the compliance
evidence trail:

```bash
gh api "orgs/<org>/audit-log?phrase=created:>2026-04-01" \
  --paginate > audit-log-2026-Q2.json
```

This is gold for "who did what" reconstruction during an incident.

## Authoritative references

- GitHub security best practices:
  <https://docs.github.com/en/code-security/getting-started/github-security-features>
- GitHub Advanced Security:
  <https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security>
- Branch protection rules:
  <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches>
- Rulesets:
  <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets>
- Secret scanning + push protection:
  <https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning>
