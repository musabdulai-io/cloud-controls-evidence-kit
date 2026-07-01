# Controls Map

The 30 controls that come up on the vast majority of customer security questionnaires, SOC 2
readiness assessments, and compliance-platform checks. Use this as the index of your evidence
folder.

For each control: the evidence required, where it lives, who owns it, and how often to refresh.

> **Tip:** delete rows for controls that don't apply (e.g. AI workload rows if you ship no AI
> features), and add rows for systems unique to your stack. The structure matters more than the
> exact list.

## 1. Access controls

| #   | Control                                    | Evidence                                                | Where to find                                                    | Owner         | Cadence         |
| --- | ------------------------------------------ | ------------------------------------------------------- | ---------------------------------------------------------------- | ------------- | --------------- |
| 1.1 | MFA enforced for all admin users           | IdP MFA policy export + per-user enforcement screenshot | Okta / Google Workspace / Microsoft Entra → Policies             | IT / Security | Quarterly       |
| 1.2 | Least-privilege IAM roles                  | IAM role definitions + per-role member list             | AWS IAM / GCP IAM / Azure AAD                                    | Platform      | Quarterly       |
| 1.3 | Service-account / long-lived key inventory | Inventory + rotation policy + last-rotated timestamps   | AWS IAM access keys / GCP IAM service-account keys / GitHub PATs | Platform      | Quarterly       |
| 1.4 | Offboarding revokes all access             | Offboarding checklist + 3–5 completed runs              | Internal runbook + IdP audit log + Cloud audit logs              | HR / IT       | Per offboarding |
| 1.5 | Privileged access review                   | Quarterly review minutes + sign-off                     | Internal doc + IdP membership export                             | Security      | Quarterly       |

## 2. Logging and monitoring

| #   | Control                                                    | Evidence                                                              | Where to find                                                             | Owner    | Cadence   |
| --- | ---------------------------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------- | -------- | --------- |
| 2.1 | Cloud audit logging enabled across all production accounts | Trail / log-sink config export covering every prod account or project | AWS CloudTrail Org Trail / GCP Cloud Audit Logs sink / Azure Activity Log | Platform | Quarterly |
| 2.2 | Audit log retention ≥ 365 days for the review period       | Storage lifecycle policy + sample query showing oldest available log  | S3 / GCS / Storage Account lifecycle rules                                | Platform | Annually  |
| 2.3 | Application logs collected centrally                       | Logging config + sample query                                         | Datadog / Honeycomb / Grafana Loki / native                               | Platform | Annually  |
| 2.4 | Security alerts route to on-call                           | Alert configuration + sample firing                                   | PagerDuty / Opsgenie / native cloud alerts                                | Security | Annually  |
| 2.5 | Tamper-resistant log storage                               | Object-lock / WORM / immutability config                              | S3 Object Lock / GCS Bucket Lock                                          | Platform | Annually  |

## 3. CI/CD and change management

| #   | Control                                  | Evidence                                                   | Where to find                  | Owner             | Cadence                |
| --- | ---------------------------------------- | ---------------------------------------------------------- | ------------------------------ | ----------------- | ---------------------- |
| 3.1 | Required reviews on production deploy    | Branch-protection config + CODEOWNERS                      | GitHub / GitLab branch rules   | Engineering leads | Quarterly              |
| 3.2 | Environment-protected production deploys | GitHub Environments / GitLab protected environments config | Repo → Settings → Environments | Engineering leads | Quarterly              |
| 3.3 | Signed / verified deploy artifacts       | Sigstore / SLSA attestation samples (when adopted)         | CI build logs                  | Platform          | Quarterly              |
| 3.4 | Release evidence: who shipped what, when | Last 30 production deploys with author + approval trail    | CI/CD history + PR merges      | Engineering leads | Per deploy / quarterly |
| 3.5 | Rollback capability documented           | Runbook + last rollback exercise                           | Internal docs                  | Platform          | Annually               |

## 4. Vulnerability and secrets hygiene

| #   | Control                                          | Evidence                                                                    | Where to find                            | Owner    | Cadence   |
| --- | ------------------------------------------------ | --------------------------------------------------------------------------- | ---------------------------------------- | -------- | --------- |
| 4.1 | Dependency scanning enabled with SLA on critical | Scanner config (Dependabot / Renovate / Snyk) + remediation log             | Repo settings + security advisories      | Platform | Quarterly |
| 4.2 | Container / image scanning                       | Scanner config + last 30 deploys' scan results                              | Trivy / Artifact Registry / ECR scanning | Platform | Quarterly |
| 4.3 | Secret scanning enabled, blocking on detection   | GitHub Advanced Security / GitLab secret detection config + remediation log | Repo security tab                        | Security | Quarterly |
| 4.4 | Static analysis / code scanning (SAST)           | SAST config + findings triage log                                           | GitHub CodeQL / GitLab SAST / Semgrep    | Platform | Quarterly |
| 4.5 | Pen-test or red-team within 12 months            | Engagement report or summary                                                | Internal doc / vendor portal             | Security | Annually  |

## 5. Backups, recovery, and availability

| #   | Control                                            | Evidence                                                      | Where to find                   | Owner             | Cadence   |
| --- | -------------------------------------------------- | ------------------------------------------------------------- | ------------------------------- | ----------------- | --------- |
| 5.1 | Production databases backed up                     | Backup configuration + last 7 backup success timestamps       | RDS / Cloud SQL / native        | Platform / SRE    | Quarterly |
| 5.2 | Backup restore tested                              | Restore-test runbook + last test report (row counts, runtime) | Internal doc                    | Platform / SRE    | Quarterly |
| 5.3 | Incident runbook exists                            | Runbook + last incident's postmortem                          | Internal docs / Notion          | Engineering leads | Annually  |
| 5.4 | Status page or customer-facing availability signal | Public status page URL + incident history                     | StatusPage / Atlassian / native | Platform          | Annually  |

## 6. AI workload controls

| #   | Control                           | Evidence                                                                     | Where to find                                              | Owner         | Cadence   |
| --- | --------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------- | ------------- | --------- |
| 6.1 | Model endpoints behind auth       | Auth config (API gateway / VPC / IAM) + sample request denied without token  | Vertex AI / OpenAI org settings / Azure OpenAI             | AI / Platform | Quarterly |
| 6.2 | Prompt / tool-call logging policy | Policy doc covering what's logged, retention, and access                     | Internal doc                                               | AI / Security | Annually  |
| 6.3 | Per-tenant or per-org spend caps  | Quota / rate-limit config + spend alert routing                              | Vertex AI quotas / OpenAI org limits / custom rate-limiter | AI / Platform | Quarterly |
| 6.4 | RAG document access boundaries    | Tenant-isolation policy + index/namespace mapping                            | Internal doc + vector DB config                            | AI            | Quarterly |
| 6.5 | AI data retention reviewed        | Retention policy aligned to customer contracts + DPA terms                   | Internal doc                                               | AI / Legal    | Annually  |
| 6.6 | Abuse / misuse handling           | Runbook for prompt-injection reports, content abuse, leaked-prompt incidents | Internal doc                                               | AI / Security | Annually  |

## How to use this table

1. **For each row that applies to your stack**, create a corresponding evidence file in your
   `evidence-folder-template/<category>/` folder. Use the templates in this kit as starting points.
2. **For each customer security questionnaire**, scan the questions and pull from this table to find
   the right evidence to attach. The `questionnaire-answer-examples.md` file has template answers
   that reference the standard evidence categories.
3. **At the quarterly review cadence**, walk the table and refresh anything overdue. Many controls
   drift quietly (key rotation, deploy approver lists, MFA exceptions); a quarterly walk catches
   them.

## Mapping to common frameworks

| This kit                              | SOC 2 Trust Service Criterion                 | NIST CSF                     |
| ------------------------------------- | --------------------------------------------- | ---------------------------- |
| 1.x Access controls                   | CC6.1, CC6.2, CC6.3                           | PR.AC                        |
| 2.x Logging and monitoring            | CC7.2, CC7.3                                  | DE.CM, DE.AE                 |
| 3.x CI/CD and change management       | CC8.1                                         | PR.IP-3                      |
| 4.x Vulnerability and secrets hygiene | CC7.1                                         | PR.IP-12, DE.CM-8            |
| 5.x Backups, recovery, availability   | CC9.1, A1.2                                   | RC.RP, PR.IP-4               |
| 6.x AI workload controls              | (emerging — currently scoped under CC6 / CC7) | (emerging — see NIST AI RMF) |

These mappings are approximate; your auditor or compliance platform will have the authoritative
cross-walk.
