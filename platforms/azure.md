# Azure evidence guide

Per-control reference for gathering compliance evidence on Microsoft Azure.

## 1. Access controls

### MFA enforcement (control 1.1)

**Console**: Entra ID → Security → Conditional Access → look for a policy requiring MFA on the
relevant role/group.

**CLI**:

```bash
# Conditional Access policies via Microsoft Graph (needs Policy.Read.All)
az rest --method GET \
  --url https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies

# Sign-in logs are NOT in the Activity Log (that's ARM resource operations).
# Pull them from Microsoft Graph (needs AuditLog.Read.All + Directory.Read.All).
# Single quotes keep the shell from expanding the $top OData parameter.
az rest --method GET \
  --url 'https://graph.microsoft.com/v1.0/auditLogs/signIns?$top=100'
```

Alternatively, if sign-in logs are exported to a Log Analytics workspace, query the `SigninLogs`
table with KQL:

```bash
az monitor log-analytics query --workspace <workspace-id> \
  --analytics-query "SigninLogs | where TimeGenerated > ago(1d) | project UserPrincipalName, AuthenticationRequirement, ResultType | take 100"
```

**Evidence**: the Conditional Access policy export — JSON of the rule requiring MFA — plus a sample
sign-in log entry showing MFA was satisfied.

### Privileged Identity Management (PIM)

If using PIM (recommended for production), evidence is the eligible vs active assignment list:

```bash
# Eligible role assignments
az rest --method GET --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleEligibilityScheduleInstances"

# Active role assignments
az rest --method GET --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignmentScheduleInstances"
```

PIM-based just-in-time elevation is strong evidence on its own.

### Service principals + secrets (control 1.3)

```bash
# All service principals (filter to app SPs, not enterprise apps)
az ad sp list --filter "servicePrincipalType eq 'Application'" --query "[].{appId:appId, displayName:displayName, accountEnabled:accountEnabled}"

# Credentials per service principal
az ad sp credential list --id <sp-id>
```

**Better long-term**: Managed Identities (no secrets) for Azure workloads; OIDC federation for
external CI/CD.

## 2. Logging and monitoring

### Azure Activity Log + Diagnostic Settings (control 2.1)

**Subscription-level Activity Log** is on by default; what you need to verify is that it's exported
to durable storage:

```bash
# Diagnostic settings on a subscription
az monitor diagnostic-settings subscription list \
  --subscription <subscription-id>

# Diagnostic settings on a specific resource
az monitor diagnostic-settings list --resource <resource-id>
```

For org-wide capture, use Azure Policy with the "Audit Diagnostic Settings" policy and enforce it
across all subscriptions in a management group.

### Retention (control 2.2)

If exporting to a Storage Account:

```bash
az storage account blob-service-properties show \
  --account-name <storage-account>

# Retention policies (legal hold + time-based)
az storage container immutability-policy show \
  --account-name <storage-account> --container <container>
```

For Log Analytics workspace retention:

```bash
az monitor log-analytics workspace show \
  --resource-group <rg> --workspace-name <name> \
  --query "retentionInDays"
```

### Microsoft Defender for Cloud (control 2.4 + 4.x)

The Azure equivalent of GuardDuty / SCC. Enable the plans you need (Servers, Containers, Storage,
etc.) and use Defender recommendations as compliance evidence.

```bash
az security pricing list
az security alert list
```

## 3. Change management

For ARM / Bicep deployments, the deployment history is your audit trail:

```bash
az deployment sub list --query "[].{name:name, timestamp:properties.timestamp, principalName:properties.principalId}" --output table
```

For GitHub-based deploys, see `github.md`.

## 5. Backups, recovery

### Azure SQL backups (control 5.1)

```bash
# Short-term retention — point-in-time recovery window, in days (1–35)
az sql db str-policy show --resource-group <rg> --server <server> --name <db>

# Long-term retention configuration (weekly / monthly / yearly)
az sql db ltr-policy show --resource-group <rg> --server <server> --name <db>

# Backup storage redundancy (LRS / ZRS / GRS)
az sql db show --resource-group <rg> --server <server> --name <db> \
  --query "currentBackupStorageRedundancy"

# List available long-term-retention backups (ltr-backup uses --location, not --resource-group)
az sql db ltr-backup list --location <region> --server <server> --database <db>
```

### Azure Backup (multi-service)

```bash
az backup vault list
az backup item list --resource-group <rg> --vault-name <vault>
az backup recoverypoint list --resource-group <rg> --vault-name <vault> --container-name <c> --item-name <item>
```

## 6. AI workload controls

### Azure OpenAI (control 6.1, 6.3)

```bash
# Resource-level network restrictions
az cognitiveservices account show \
  --name <resource> --resource-group <rg> \
  --query "properties.networkAcls"

# Per-deployment quota
az cognitiveservices account deployment list \
  --name <resource> --resource-group <rg>
```

**Azure OpenAI content filters** are configured per-deployment: evidence is the deployment's
`raiPolicy` setting.

### Azure budget alerts

```bash
az consumption budget list --resource-group <rg>
```

## Authoritative references

- Azure Security Benchmark v3: <https://learn.microsoft.com/en-us/security/benchmark/azure/overview>
- Microsoft Cloud Security Benchmark: <https://learn.microsoft.com/en-us/security/benchmark/cloud/>
- CIS Microsoft Azure Foundations Benchmark: <https://www.cisecurity.org/benchmark/azure>
- Azure Policy compliance:
  <https://learn.microsoft.com/en-us/azure/governance/policy/concepts/regulatory-compliance>
- Azure OpenAI responsible AI:
  <https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/content-filter>
