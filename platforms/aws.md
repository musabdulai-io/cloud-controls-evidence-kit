# AWS evidence guide

Per-control reference for gathering compliance evidence on AWS. For each control in
`../controls-map.md`, this guide tells you the AWS console path, the CLI command, and the canonical
export that satisfies an auditor or customer security review.

## 1. Access controls

### MFA enforcement (control 1.1)

**Console**: IAM → Account settings → Multi-factor authentication status (for root). For users: IAM
→ Users → filter by `MFA active = No`. For Identity Center / IAM Identity Center: Identity Center →
Settings → Authentication → MFA.

**CLI**:

```bash
# All users + their MFA status
aws iam list-users --query 'Users[].[UserName,Arn]' --output table

# Per-user MFA devices
for user in $(aws iam list-users --query 'Users[].UserName' --output text); do
  echo "$user: $(aws iam list-mfa-devices --user-name "$user" --query 'MFADevices[].SerialNumber' --output text)"
done

# Credentials report (most comprehensive — includes MFA flag per user)
aws iam generate-credential-report
aws iam get-credential-report --query Content --output text | base64 -d > credential-report.csv
```

**Evidence to save**: the credential report CSV — it's the single-page proof that lists every user,
last activity, and MFA status.

### IAM roles and least privilege (control 1.2)

**Console**: IAM → Roles → review each role's policies + trust relationship.

**CLI**:

```bash
# Full IAM authorization details for the account
aws iam get-account-authorization-details > iam-authz-details.json

# Just policies attached to a specific role
aws iam list-attached-role-policies --role-name <role>
aws iam list-role-policies --role-name <role>  # inline
```

**Evidence**: `iam-authz-details.json`. This is comprehensive but large; for a digestible version,
also keep a written summary of the privileged roles.

### Service-account / access-key inventory (control 1.3)

**CLI**:

```bash
# All IAM users + their access keys
for user in $(aws iam list-users --query 'Users[].UserName' --output text); do
  aws iam list-access-keys --user-name "$user"
done > access-key-inventory.json

# Stale keys (>90 days). CUTOFF handles both GNU/Linux and macOS/BSD `date`.
CUTOFF=$(date -d '90 days ago' +%Y-%m-%d 2>/dev/null || date -v-90d +%Y-%m-%d)
aws iam get-credential-report \
  --query Content --output text | base64 -d | \
  awk -F, -v cutoff="$CUTOFF" 'NR==1 || ($10 == "true" && $11 < cutoff)'
```

**Evidence**: `access-key-inventory-<date>.json` + documented rotation policy.

**Better long-term**: replace IAM users + access keys with IAM Identity Center (SSO) for humans and
IAM Roles for Service Accounts (EKS/Lambda/ECS service-linked roles) for workloads.

## 2. Logging and monitoring

### CloudTrail organization trail (control 2.1)

**Console**: CloudTrail → Trails → look for `IsOrganizationTrail = Yes`, `IsMultiRegionTrail = Yes`.

**CLI**:

```bash
# Confirm an org-level multi-region trail exists
aws cloudtrail describe-trails \
  --query 'trailList[?IsOrganizationTrail==`true` && IsMultiRegionTrail==`true`]'

# Confirm it's actively logging
aws cloudtrail get-trail-status --name <trail-name>

# Confirm management + data events are configured
aws cloudtrail get-event-selectors --trail-name <trail-name>
```

**Evidence**: `cloudtrail-org-config.json` + `cloudtrail-status.json`.

### CloudTrail retention + tamper resistance (control 2.2 + 2.5)

**CLI**:

```bash
# S3 bucket lifecycle (where CloudTrail logs land)
aws s3api get-bucket-lifecycle-configuration --bucket <cloudtrail-bucket>

# Object Lock (tamper resistance)
aws s3api get-object-lock-configuration --bucket <cloudtrail-bucket>

# Sample query for oldest log (via Athena over CloudTrail)
# Replace <db> and <table> with your Athena CloudTrail config
# SELECT MIN(eventtime) FROM <db>.<table>;
```

**Evidence**: lifecycle policy + object-lock configuration. Use Object Lock in **Compliance** mode
(not Governance) for stronger tamper resistance — even the root user can't override.

### GuardDuty / Security Hub (control 2.4)

Optional but credibility-boosting:

```bash
# GuardDuty: enabled + active in each region
aws guardduty list-detectors
aws guardduty get-detector --detector-id <id>

# Security Hub findings
aws securityhub describe-hub
aws securityhub get-findings --max-results 10
```

## 4. Vulnerability and image scanning

### ECR image scanning (control 4.2)

**CLI**:

```bash
# Per-repo scanning setting (scanOnPush lives on the repository object —
# there is no `describe-image-scanning-configuration` command)
aws ecr describe-repositories \
  --query 'repositories[].{name:repositoryName,scanOnPush:imageScanningConfiguration.scanOnPush}'

# Effective scanning configuration per repo (basic vs Enhanced/Inspector v2)
aws ecr batch-get-repository-scanning-configuration --repository-names <repo>

# Scan findings for a specific image
aws ecr describe-image-scan-findings \
  --repository-name <repo> \
  --image-id imageDigest=<digest>
```

**Better**: use ECR Enhanced Scanning (Amazon Inspector v2) for continuous, deeper scans — set it
once at the registry level with `aws ecr put-registry-scanning-configuration`.

Required IAM: `ecr:DescribeRepositories`, `ecr:BatchGetRepositoryScanningConfiguration`,
`ecr:DescribeImageScanFindings`. Docs:
<https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html>

## 5. Backups, recovery

### RDS automated backups (control 5.1)

**CLI**:

```bash
aws rds describe-db-instances --db-instance-identifier <id> \
  --query 'DBInstances[0].{BackupRetentionPeriod:BackupRetentionPeriod,PreferredBackupWindow:PreferredBackupWindow,DBInstanceStatus:DBInstanceStatus,StorageEncrypted:StorageEncrypted}'

# Backup history
aws rds describe-db-snapshots \
  --db-instance-identifier <id> \
  --snapshot-type automated --max-records 30
```

**Evidence**: backup-retention config + last 7 days of successful snapshots.

### AWS Backup (multi-service)

If using AWS Backup:

```bash
aws backup list-backup-plans
aws backup get-backup-plan --backup-plan-id <id>
aws backup list-recovery-points-by-backup-vault --backup-vault-name <vault>
```

## 6. AI workload controls

If running on AWS Bedrock:

```bash
# Per-account model access policies
aws bedrock list-foundation-models

# Bedrock guardrails (content filtering, PII detection)
aws bedrock list-guardrails
aws bedrock get-guardrail --guardrail-identifier <id>

# CloudWatch billing alarm for Bedrock spend
aws cloudwatch describe-alarms --alarm-name-prefix "bedrock-spend"
```

## Authoritative references

- AWS Well-Architected Security Pillar:
  <https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html>
- AWS Foundational Security Best Practices (Security Hub):
  <https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp.html>
- CIS AWS Foundations Benchmark: <https://www.cisecurity.org/benchmark/amazon_web_services>
- AWS Audit Manager (for SOC 2, ISO 27001 evidence mapping):
  <https://docs.aws.amazon.com/audit-manager/>
