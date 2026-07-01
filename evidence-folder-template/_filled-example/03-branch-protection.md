# Branch protection — Acme Analytics

> Illustrative example for a fictional company. See the blank template at
> `../03-cicd-change-management/branch-protection.md`.

Production deploys to Acme's services require a reviewed, merged pull request. Protection is applied
via an **org-level GitHub ruleset**, so a newly created repo inherits it automatically rather than
shipping with an unprotected `main`.

## Evidence on file

| Artifact                                | What it shows                                                              |
| --------------------------------------- | -------------------------------------------------------------------------- |
| `org-ruleset-2026-06-18.json`           | Org ruleset: require PR, ≥1 approval, dismiss stale reviews, no force-push |
| `branch-protection-app-2026-06-18.json` | Per-repo protection on the primary `app` repo (belt + suspenders)          |
| `CODEOWNERS-2026-06-18.txt`             | CODEOWNERS routing review to the owning team per path                      |

## Gathered from

```bash
gh api orgs/acme/rulesets > org-rulesets.json
gh api orgs/acme/rulesets/4412 > org-ruleset-2026-06-18.json
gh api repos/acme/app/branches/main/protection > branch-protection-app-2026-06-18.json
gh api repos/acme/app/contents/.github/CODEOWNERS --jq '.content' | base64 -d > CODEOWNERS-2026-06-18.txt
```

## Buyer questions answered

> "Is code reviewed before reaching production?"

Yes. The org ruleset requires a pull request with at least one approving review and dismisses stale
approvals when new commits are pushed. Direct pushes and force-pushes to `main` are blocked for
everyone, including admins (`enforcement: active`, no bypass actors).

> "How do you ensure new repositories are protected?"

The ruleset is org-level (`target: branch`, applied to all repos), so protection is inherited by
default — adding a repo does not create an unprotected production branch.

## Refresh

Quarterly. Last refreshed 2026-06-18 by R. Mensah. Next due 2026-09-18.
