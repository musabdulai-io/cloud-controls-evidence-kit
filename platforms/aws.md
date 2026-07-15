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
#!/usr/bin/env bash
# Access-key inventory + stale-key report. Fails closed: every step that could
# leave you holding an empty file aborts instead.
set -euo pipefail    # -o pipefail matters: without it a failed `aws` upstream of
                     # a pipe still exits 0 and you file an empty artifact.

# 1. All IAM users + their access keys, as ONE valid JSON array (a bare loop
#    concatenates separate JSON documents, which jq and most tooling reject).
aws iam list-users --query 'Users[].UserName' --output text \
  | tr '\t' '\n' \
  | while read -r user; do
      aws iam list-access-keys --user-name "$user" \
        --query 'AccessKeyMetadata[].{user:UserName,id:AccessKeyId,status:Status,created:CreateDate}'
    done \
  | jq -s 'add // []' > access-key-inventory.json

# 2. The credential report is generated ASYNCHRONOUSLY. generate- returns
#    STARTED/INPROGRESS/COMPLETE, and get- raises ReportInProgress until it's
#    ready. Poll to COMPLETE before reading, or you can end up with nothing.
for _ in $(seq 1 30); do
  state=$(aws iam generate-credential-report --query State --output text)
  [ "$state" = "COMPLETE" ] && break
  sleep 2
done
[ "$state" = "COMPLETE" ] || { echo "credential report not ready (state=$state)" >&2; exit 1; }

# 3. Download to a real file, then VALIDATE it before filtering. An empty or
#    header-less file must abort, not quietly become "no stale keys".
report=$(mktemp)
trap 'rm -f "$report"' EXIT
aws iam get-credential-report --query Content --output text | base64 -d > "$report"
[ -s "$report" ] || { echo "credential report is empty" >&2; exit 1; }
head -1 "$report" | grep -q 'access_key_1_active' \
  || { echo "unexpected credential-report format — refusing to filter it" >&2; exit 1; }

# 4. Only now filter. Parsed by COLUMN NAME, not position: the report has two key
#    slots (access_key_1_*, access_key_2_*) and a positional expression that
#    checks the wrong column silently reports "no stale keys" instead of failing.
CUTOFF=$(date -d '90 days ago' +%Y-%m-%d 2>/dev/null || date -v-90d +%Y-%m-%d)
python3 -c '
import csv, sys
cutoff = sys.argv[1]
with open(sys.argv[2], newline="") as fh:
    for row in csv.DictReader(fh):
        for slot in ("access_key_1", "access_key_2"):   # BOTH slots, not just the first
            if row[slot + "_active"] != "true":
                continue
            rotated = row[slot + "_last_rotated"]
            if rotated in ("N/A", "not_supported", ""):
                continue
            if rotated[:10] < cutoff:
                print("\t".join([row["user"], slot, rotated[:10]]))
' "$CUTOFF" "$report" > stale-access-keys.tsv

echo "stale keys: $(wc -l < stale-access-keys.tsv)"
```

`root_account` appears as a row like any other; it should have no active keys at all, so if it shows
up here, that is the finding.

**Sanity-check the parse before filing an empty result as evidence.** "No stale keys" and "the
filter matched nothing" look identical in an output file, and only one of them is evidence:

```bash
# Should print every user with at least one ACTIVE key. If this is empty but you
# know keys exist, the parse is wrong — don't file the empty stale-key result.
python3 -c '
import csv, sys
for row in csv.DictReader(open(sys.argv[1], newline="")):
    slots = [s for s in ("access_key_1", "access_key_2") if row[s + "_active"] == "true"]
    if slots:
        print(row["user"], slots)
' "$report"
```

**References**:
[GenerateCredentialReport](https://docs.aws.amazon.com/IAM/latest/APIReference/API_GenerateCredentialReport.html)
(async states) ·
[GetCredentialReport](https://docs.aws.amazon.com/IAM/latest/APIReference/API_GetCredentialReport.html)
(ReportInProgress) ·
[credential report field definitions](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_getting-report.html)
— confirm the column names there rather than assuming an order.

**Evidence**: `access-key-inventory-<date>.json` + `stale-access-keys-<date>.tsv` + documented
rotation policy.

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
