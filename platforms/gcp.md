# GCP evidence guide

Per-control reference for gathering compliance evidence on Google Cloud Platform.

## 1. Access controls

### MFA enforcement (control 1.1)

**Workspace admin console**: Admin → Security → 2-step verification → Enforcement. Cloud Identity
admins are tied to Workspace; enforce 2SV there.

**CLI** (Cloud Identity / Workspace): there is no `gcloud` command that lists per-user 2SV status —
the Admin SDK Directory API is the source of truth (the `isEnrolledIn2Sv` / `isEnforcedIn2Sv` fields
on the `users` resource; these are **not** on the Cloud Identity group-membership resource).

```bash
# Admin SDK Directory API — org users not enrolled in 2SV
curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://admin.googleapis.com/admin/directory/v1/users?customer=my_customer&fields=users(primaryEmail,isEnrolledIn2Sv,isEnforcedIn2Sv)" \
  | jq '.users[] | select(.isEnrolledIn2Sv == false) | .primaryEmail'
```

**Required access**: a Workspace / Cloud Identity admin role and the `admin.directory.user.readonly`
OAuth scope. If you prefer a CLI wrapper, [GAM](https://github.com/GAM-team/GAM) calls the same
Admin SDK API (e.g. `gam print users fields primaryEmail,isenrolledin2sv`). For dedicated Cloud
Identity (no Workspace), the Admin SDK API is still the source of truth. Reference:
<https://developers.google.com/workspace/admin/directory/reference/rest/v1/users>

### IAM roles + least privilege (control 1.2)

**CLI**:

```bash
# Project-level IAM policy
gcloud projects get-iam-policy <project-id> --format=json > iam-policy.json

# Org-level IAM
gcloud organizations get-iam-policy <org-id> --format=json > org-iam-policy.json

# Asset Inventory: comprehensive snapshot of all IAM across projects
gcloud asset search-all-iam-policies --scope=organizations/<org-id> > org-iam-asset.json
```

**Evidence**: the asset-inventory IAM export is the most comprehensive single artifact.

### Service-account keys (control 1.3)

**CLI**:

```bash
# All service accounts in a project
gcloud iam service-accounts list --project=<project>

# Keys per service account (look for USER_MANAGED — those are long-lived)
for sa in $(gcloud iam service-accounts list --project=<project> --format='value(email)'); do
  gcloud iam service-accounts keys list --iam-account="$sa" \
    --filter="keyType=USER_MANAGED" --format=json
done > user-managed-keys.json
```

**The right answer is Workload Identity Federation**, not key rotation. WIF lets workloads (GKE,
Cloud Run, Cloud Build, even external systems via OIDC) authenticate without long-lived keys.

```bash
# Inventory of workload identity pools
gcloud iam workload-identity-pools list --location=global
```

## 2. Logging and monitoring

### Cloud Audit Logs sink (control 2.1)

**Console**: Logging → Log Router → look for an org-level sink exporting to BigQuery, GCS, or
Pub/Sub.

**CLI**:

```bash
# Org-level sinks
gcloud logging sinks list --organization=<org-id>
gcloud logging sinks describe <sink-name> --organization=<org-id>

# Data Access logs configuration per project (these are NOT on by default)
gcloud projects get-iam-policy <project> --format='value(auditConfigs)'
```

**Important**: Admin Activity and System Event logs are always on and free. **Data Access logs are
off by default** and must be explicitly enabled. Customer security reviews ask about Data Access
logs specifically.

### Retention + tamper resistance (control 2.2 + 2.5)

For logs sinked to GCS:

```bash
# Bucket retention policy
gsutil retention get gs://<bucket>

# Retention lock — once locked, even admins can't shorten retention
# Lock it after confirming the retention is right
gsutil retention lock gs://<bucket>
```

For logs sinked to BigQuery: set table partitioning expiration to your retention window.

### Cloud Monitoring alerts (control 2.4)

```bash
# Alert policies
gcloud alpha monitoring policies list --format=json

# Notification channels (where do alerts go?)
gcloud beta monitoring channels list
```

**Evidence**: alert policies + notification channels mapped to your on-call rotation.

## 4. Vulnerability scanning

### Container Analysis (control 4.2)

**Enable**:

```bash
gcloud services enable containeranalysis.googleapis.com
gcloud services enable containerscanning.googleapis.com
```

**Query results**:

```bash
# Vulnerabilities for a specific image
gcloud artifacts docker images describe <image-uri> --show-package-vulnerability

# Org-wide via Asset Inventory
gcloud asset search-all-resources \
  --scope=organizations/<org-id> \
  --asset-types=containeranalysis.googleapis.com/Occurrence
```

### Security Command Center (control 4.x + 2.4)

GCP's equivalent of AWS Security Hub. Enable Standard tier (free) or Premium for richer scanning.
Findings are credible evidence on their own.

```bash
gcloud scc findings list <org-id>
```

## 5. Backups, recovery

### Cloud SQL backups (control 5.1)

```bash
# Backup config
gcloud sql instances describe <instance> \
  --format='yaml(settings.backupConfiguration)'

# Backup history (last 10)
gcloud sql backups list --instance=<instance> --limit=10
```

### Cloud Storage versioning + object lifecycle (control 5.1)

```bash
gsutil versioning get gs://<bucket>
gsutil lifecycle get gs://<bucket>
```

## 6. AI workload controls

### Vertex AI quotas + endpoints (control 6.1, 6.3)

```bash
# List deployed models
gcloud ai endpoints list --region=<region>
gcloud ai models list --region=<region>

# Endpoint config — confirm auth is required
gcloud ai endpoints describe <endpoint-id> --region=<region>

# Per-project quotas
gcloud alpha services quota list --service=aiplatform.googleapis.com --consumer=projects/<project>
```

**Vertex AI Safety Filters** (for generative models): apply them at deploy time and document the
configuration.

### Cloud Billing budgets + spend caps

```bash
# Billing budgets
gcloud billing budgets list --billing-account=<id>
```

Set budget alerts at 50/80/100% of monthly cap. Prefer soft controls first: per-service quotas, API
gateway / rate-limit caps, alerting, and workload-level throttles. A true hard cap (a Pub/Sub →
Cloud Function that disables billing) is a **last-resort kill switch** — disabling billing can take
down every workload in the project, so reserve it for noncritical/sandbox projects, never
production.

## Authoritative references

- Google Cloud Security Best Practices: <https://cloud.google.com/security/best-practices>
- CIS Google Cloud Platform Foundations Benchmark:
  <https://www.cisecurity.org/benchmark/google_cloud_computing_platform>
- Cloud Audit Logs overview: <https://cloud.google.com/logging/docs/audit>
- Compliance Reports Manager: <https://cloud.google.com/security/compliance>
- Vertex AI Responsible AI guidelines:
  <https://cloud.google.com/vertex-ai/docs/generative-ai/learn/responsible-ai>
