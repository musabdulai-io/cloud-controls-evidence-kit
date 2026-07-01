# Evidence register

The evidence register is the index of your whole compliance program: one row per control, pointing
at the artifact that proves it. When an auditor sends a PBC (Provided By Client) list or a buyer
sends a security questionnaire, you answer from this table instead of starting a scavenger hunt.

Keep it in your private evidence repo next to the populated
[`evidence-folder-template/`](evidence-folder-template/README.md). The
[`controls-map.md`](controls-map.md) tells you _what_ evidence each control needs; this register
tracks the _actual artifacts you have_ and their freshness.

## Columns

| Column           | What goes here                                                                      |
| ---------------- | ----------------------------------------------------------------------------------- |
| `control`        | Control ID + name from `controls-map.md` (e.g. `1.1 MFA enforcement`)               |
| `system`         | The system/account the evidence covers (e.g. `AWS prod`, `GitHub org`, `Cloud SQL`) |
| `owner`          | Named person accountable for refreshing it (not a team — a person)                  |
| `artifact`       | The file/export that proves it (e.g. `01-access-controls/mfa-enforcement.md`)       |
| `source`         | Where it was exported from — console path, CLI command, or dashboard URL            |
| `cadence`        | How often it must be refreshed (`Quarterly`, `Annually`, `Per release`)             |
| `last_refreshed` | ISO date the artifact was last regenerated (`2026-06-30`)                           |
| `status`         | `current`, `stale`, `missing`, or `n/a`                                             |

**Status rule of thumb:** mark a row `stale` once `last_refreshed` is older than its `cadence`
window. A quick way to spot the backlog before an auditor does is to sort by `status`.

## Example (fictional Series B SaaS)

| control                 | system           | owner     | artifact                                         | source                                           | cadence   | last_refreshed | status  |
| ----------------------- | ---------------- | --------- | ------------------------------------------------ | ------------------------------------------------ | --------- | -------------- | ------- |
| 1.1 MFA enforcement     | Google Workspace | A. Okada  | `01-access-controls/mfa-enforcement.md`          | Admin → Security → 2-step verification           | Quarterly | 2026-06-12     | current |
| 1.2 Least-privilege IAM | AWS prod         | A. Okada  | `01-access-controls/iam-quarterly-review.md`     | `aws iam get-account-authorization-details`      | Quarterly | 2026-04-02     | stale   |
| 2.1 Audit logging       | AWS prod         | R. Mensah | `02-logging-monitoring/audit-log-config.md`      | CloudTrail org-trail config export               | Quarterly | 2026-06-20     | current |
| 3.1 Branch protection   | GitHub org       | R. Mensah | `03-cicd-change-management/branch-protection.md` | `gh api repos/acme/app/branches/main/protection` | Quarterly | 2026-06-18     | current |
| 4.3 Secret scanning     | GitHub org       | R. Mensah | `04-vulnerability-secrets/secret-scanning.md`    | `gh api orgs/acme --jq '.security_and_analysis'` | Quarterly | 2026-06-18     | current |
| 5.2 Restore test        | Cloud SQL        | L. Favre  | `05-backups-recovery/restore-test.md`            | Quarterly DR drill report                        | Quarterly | 2026-03-30     | stale   |
| 6.1 Model endpoint auth | Vertex AI        | L. Favre  | `06-ai-workload-controls/endpoint-auth.md`       | `gcloud ai endpoints describe <id>`              | Quarterly | —              | missing |

## CSV version

If you'd rather track this in a spreadsheet or pipe it into a script, the same schema as CSV — copy
this into `evidence-register.csv`:

```csv
control,system,owner,artifact,source,cadence,last_refreshed,status
1.1 MFA enforcement,Google Workspace,,01-access-controls/mfa-enforcement.md,Admin > Security > 2-step verification,Quarterly,,missing
1.2 Least-privilege IAM,,,01-access-controls/iam-quarterly-review.md,,Quarterly,,missing
1.3 Service-account key inventory,,,01-access-controls/iam-quarterly-review.md,,Quarterly,,missing
2.1 Audit logging,,,02-logging-monitoring/audit-log-config.md,,Quarterly,,missing
2.2 Log retention,,,02-logging-monitoring/retention-policy.md,,Annually,,missing
2.3 Alert routing,,,02-logging-monitoring/alert-routing.md,,Annually,,missing
3.1 Branch protection,,,03-cicd-change-management/branch-protection.md,,Quarterly,,missing
3.2 Deploy approvals,,,03-cicd-change-management/deploy-approvals.md,,Quarterly,,missing
3.4 Release evidence,,,03-cicd-change-management/release-evidence.md,,Quarterly,,missing
4.1 Dependency scanning,,,04-vulnerability-secrets/dependency-scanning.md,,Quarterly,,missing
4.2 Image scanning,,,04-vulnerability-secrets/image-scanning.md,,Quarterly,,missing
4.3 Secret scanning,,,04-vulnerability-secrets/secret-scanning.md,,Quarterly,,missing
5.1 Backup config,,,05-backups-recovery/backup-config.md,,Quarterly,,missing
5.2 Restore test,,,05-backups-recovery/restore-test.md,,Quarterly,,missing
5.3 Incident runbook,,,05-backups-recovery/incident-runbook.md,,Annually,,missing
6.1 Model endpoint auth,,,06-ai-workload-controls/endpoint-auth.md,,Quarterly,,missing
6.2 Prompt/tool-call logging,,,06-ai-workload-controls/prompt-tool-call-logging.md,,Annually,,missing
6.3 Spend caps,,,06-ai-workload-controls/spend-caps.md,,Quarterly,,missing
6.4 Data retention,,,06-ai-workload-controls/data-retention.md,,Annually,,missing
```

Fill in `system`, `owner`, `source`, and `last_refreshed` as you collect each artifact, and flip
`status` to `current`. Rows that stay `missing` are your readiness backlog.
