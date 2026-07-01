# AI workload spend caps — Acme Analytics

> Illustrative example for a fictional company. See the blank template at
> `../06-ai-workload-controls/spend-caps.md`.

Acme's AI features call Bedrock (production) and Vertex AI (a batch enrichment job). Both have
per-environment quotas and budget alerts so a runaway loop or abuse can't produce an unbounded bill,
and breaches page a human.

## Evidence on file

| Artifact                         | What it shows                                           |
| -------------------------------- | ------------------------------------------------------- |
| `bedrock-quotas-2026-06-20.json` | Provisioned/throughput limits + per-model access policy |
| `budget-alerts-2026-06-20.json`  | Budget thresholds (50/80/100%) + notification channels  |
| `alert-routing-2026-06-20.md`    | Where a cap breach pages (PagerDuty → on-call SRE)      |

## Gathered from

```bash
# AWS Bedrock spend alarm + service quotas
aws cloudwatch describe-alarms --alarm-name-prefix "bedrock-spend" > bedrock-alarms.json
aws service-quotas list-service-quotas --service-code bedrock > bedrock-quotas-2026-06-20.json

# GCP budget for the AI project
gcloud billing budgets list --billing-account=01ABCD-234567-89EFGH > budget-alerts-2026-06-20.json
```

## Controls in place

- **Per-model rate limits** on the production Bedrock endpoint (capped at 60 req/s, well above
  steady-state, low enough to bound abuse).
- **Budget alerts** at 50 / 80 / 100% of a $4,000/month AI cap, routed to `#ai-oncall` and
  PagerDuty.
- **Hard stop**: at 100% a Lambda disables the non-critical Vertex batch job (the production
  inference path stays up; only discretionary spend is cut).

## Buyer questions answered

> "How do you prevent runaway AI spend or abuse?"

Per-model rate limits bound request volume; tiered budget alerts (50/80/100%) page the on-call SRE;
the 100% threshold automatically disables the discretionary batch workload. The most recent alert
(2026-05-30, 80% threshold) was a legitimate traffic spike and resolved without action.

## Refresh

Quarterly. Last refreshed 2026-06-20 by L. Favre. Next due 2026-09-20.
