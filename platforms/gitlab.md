# GitLab evidence guide

Per-control reference for gathering compliance evidence in GitLab. For teams on GitLab, the group
and project settings are the critical-control surface for code review, deploy approvals, and secret
detection — the GitLab equivalent of the GitHub guide.

Examples use the `glab` CLI (`glab api ...`) for parity with the GitHub guide; every call also works
as a `curl` with a `PRIVATE-TOKEN` header. Replace `<group>` / `<project>` with the numeric ID or
URL-encoded path (e.g. `mygroup%2Fmyrepo`).

## 1. Access controls

### Group membership + 2FA (control 1.1)

**UI**: Group → Settings → General → Permissions and group features → "Require all users in this
group to set up two-factor authentication".

**CLI / API**:

```bash
# Group-level 2FA enforcement flag
glab api groups/<group> --jq '.require_two_factor_authentication'

# Group members with their role. access_level: 10 Guest, 20 Reporter,
# 30 Developer, 40 Maintainer, 50 Owner. The members object DOES include
# access_level, so filtering on it is correct.
glab api "groups/<group>/members/all?per_page=100" --paginate \
  --jq '.[] | {username, access_level}'

# Group owners only (access_level 50)
glab api "groups/<group>/members/all?per_page=100" --paginate \
  --jq '[.[] | select(.access_level == 50) | .username]'
```

**Evidence**: confirm `require_two_factor_authentication = true` at the group level, plus the owner
list above. Note: the members endpoint does not expose per-user 2FA status — for that, an instance
admin uses the Users API (`/users?two_factor=disabled`, admin-only) or the GitLab UI admin area.

### Personal / project / group access tokens

Tokens are a common blind spot — a leaked token bypasses 2FA.

```bash
# Group access tokens (Maintainer+)
glab api "groups/<group>/access_tokens" \
  --jq '[.[] | {name, scopes, expires_at, active}]'

# Project access tokens
glab api "projects/<project>/access_tokens" \
  --jq '[.[] | {name, scopes, expires_at, active}]'
```

**Best practice**: scope tokens minimally, set expirations, and document any long-lived tokens with
owner + rotation date. Enforce a maximum token lifetime at the instance/group level where available.

### CODEOWNERS (control 3.1 dependency)

```bash
# Pull the canonical CODEOWNERS (raw file from the default branch)
glab api "projects/<project>/repository/files/CODEOWNERS/raw?ref=main"
```

CODEOWNERS-based approval requires Premium/Ultimate; on Free, use Maintainer-only merge access plus
approval rules.

## 3. Change management

### Protected branches (control 3.1)

```bash
# Protected branch config (push/merge access levels, force-push, code-owner approval)
glab api "projects/<project>/protected_branches" > protected-branches.json
glab api "projects/<project>/protected_branches/main" > protected-main.json
```

**Evidence**: `main` (and any release branches) protected with no direct push, merge restricted to
Maintainers or via merge request only.

### Merge request approvals (control 3.1)

```bash
# Project-level approval settings (e.g. prevent author self-approval,
# reset approvals on new commits)
glab api "projects/<project>/approvals" > mr-approval-settings.json

# Approval rules (required approvers, minimum approver count)
glab api "projects/<project>/approval_rules" > mr-approval-rules.json
```

**The control that matters**: `approvals_before_merge` ≥ 1 with
`merge_requests_author_approval = false` so authors can't approve their own MRs. Reset-approvals-
on-push closes the "approve a clean diff, then push a bad commit" gap.

### Protected environments (control 3.2)

```bash
# Deployment approvals + which roles can deploy to production
glab api "projects/<project>/protected_environments" > protected-environments.json
glab api "projects/<project>/protected_environments/production" > protected-production.json
```

### Deploy history (control 3.4)

```bash
# Recent deployments to production with status + who triggered them
glab api "projects/<project>/deployments?environment=production&per_page=30&order_by=created_at&sort=desc" \
  --jq '[.[] | {id, status, ref, user: .user.username, created_at}]' \
  > recent-deploys.json
```

### Push rules (secret / commit hygiene)

```bash
# Project push rules (e.g. prevent committing secrets, require signed commits,
# enforce author email domain). Premium+.
glab api "projects/<project>/push_rule"
```

## 4. Vulnerability + secret scanning

GitLab's scanners run as CI jobs (Ultimate for the full security dashboard). Enable by including the
managed templates in `.gitlab-ci.yml`:

```yaml
include:
  - template: Jobs/Dependency-Scanning.gitlab-ci.yml
  - template: Jobs/SAST.gitlab-ci.yml
  - template: Jobs/Secret-Detection.gitlab-ci.yml
  - template: Jobs/Container-Scanning.gitlab-ci.yml
```

**Evidence**: confirm the templates are present in the pipeline config, then export findings.

```bash
# Confirm the scanning jobs are wired into the pipeline
glab api "projects/<project>/repository/files/.gitlab-ci.yml/raw?ref=main"
```

For the findings themselves, the security dashboard (Project → Secure → Vulnerability report) and
its CSV export are the most stable evidence. GitLab is **deprecating the REST vulnerability
endpoints** in favour of GraphQL, so prefer GraphQL for automation:

```bash
glab api graphql -f query='
  query {
    project(fullPath: "<group>/<project>") {
      vulnerabilities(first: 100, state: DETECTED) {
        nodes { title severity state reportType }
      }
    }
  }'
```

**The control that matters**: Secret Detection with the pipeline configured to block on new
findings, plus a push rule that rejects commits containing secrets — so the secret never reaches the
repository in the first place.

## Audit events

GitLab keeps an audit-events stream (Premium for group-level, Ultimate / self-managed for the full
instance log). Pull this for the "who did what" compliance trail:

```bash
# Group audit events (Premium+)
glab api "groups/<group>/audit_events?created_after=2026-04-01&per_page=100" \
  --paginate > audit-events-2026-Q2.json
```

This is the evidence trail for access changes, permission grants, and setting changes during an
incident reconstruction.

## Authoritative references

- GitLab security best practices: <https://docs.gitlab.com/security/>
- Protected branches: <https://docs.gitlab.com/user/project/repository/branches/protected/>
- Merge request approvals: <https://docs.gitlab.com/user/project/merge_requests/approvals/>
- Protected environments: <https://docs.gitlab.com/ci/environments/protected_environments/>
- Enforce 2FA for a group: <https://docs.gitlab.com/security/two_factor_authentication/>
- Secret detection: <https://docs.gitlab.com/user/application_security/secret_detection/>
- Audit events: <https://docs.gitlab.com/administration/audit_event_reports/>
- Members API (`access_level` values): <https://docs.gitlab.com/api/members/>
