# Restore test — Acme Analytics

> Illustrative example for a fictional company. See the blank template at
> `../05-backups-recovery/restore-test.md`.

Acme runs a quarterly restore from the production Cloud SQL backups into an isolated environment,
measures the outcome, and files the report. This is the artifact most teams lack and most buyers ask
for.

## Evidence on file

| Artifact                     | What it shows                        |
| ---------------------------- | ------------------------------------ |
| `restore-test-2026-06-14.md` | The completed Q2 test report (below) |
| `restore-runbook.md`         | The procedure followed               |

## Q2 restore test report

**Tester:** L. Favre · **Reviewer:** R. Mensah

| Item                | Value                                           |
| ------------------- | ----------------------------------------------- |
| Source backup       | Automated snapshot 2026-06-13T03:00Z            |
| Source DB           | prod-main (Cloud SQL Postgres 16)               |
| Target environment  | restore-test (same region, isolated VPC)        |
| Restore initiated   | 2026-06-14T14:00Z                               |
| Restore completed   | 2026-06-14T14:41Z                               |
| **Wall-clock time** | **41 min**                                      |
| Row count: users    | 211,904 (expected ~211,900; delta 0.002%)       |
| Row count: events   | 8,442,118 (expected ~8,442,000; delta 0.001%)   |
| FK integrity        | PASS                                            |
| Application boot    | PASS — read-only smoke test against restored DB |
| Tear-down           | Completed 2026-06-14T15:05Z                     |

**Outcome:** PASS. Restore time 41 min, well within RTO (4 h). No integrity issues.

## RPO / RTO answer

> **RPO**: 5 minutes — point-in-time recovery is enabled; worst-case data loss is one WAL interval.
>
> **RTO**: 4 hours — validated by the quarterly restore test (most recent 2026-06-14, 41 min
> wall-clock, comfortably under target).

## Refresh

Quarterly. Last refreshed 2026-06-14 by L. Favre. Next due 2026-09-14.
