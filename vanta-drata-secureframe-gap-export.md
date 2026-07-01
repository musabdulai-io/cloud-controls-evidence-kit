# Turning compliance-platform gaps into a fix queue

Vanta, Drata, and Secureframe are excellent at _continuous monitoring_ — they connect to your cloud
and SaaS accounts and surface every control that's currently failing. What they don't do is _fix_
anything. A red "MFA not enforced" or "branch protection missing" check sits there until an engineer
changes the config, gathers the evidence, and closes the loop.

This guide turns the platform's failing-check list into a prioritized engineering queue. (For the
broader argument, see
[Vanta and Drata Show the Gaps. Who Actually Fixes Them?](https://musabdulai.com/resources/vanta-drata-gaps-who-fixes-them))

## 1. Export the failing checks

Each platform lets you export controls/tests and their status. Pull the failing set:

- **Vanta** — Tests page → filter to **Failing** / **Needs attention** → export, or use the
  [Vanta API](https://developer.vanta.com/) `tests`/`test-entities` endpoints.
- **Drata** — Controls / Monitors → filter to **Failing** / **Unhealthy** → export CSV, or the
  [Drata Public API](https://developers.drata.com/) controls endpoint.
- **Secureframe** — Tests → filter to **Failing** → export, or use the Secureframe API.

You want, per failing item: the check name, the control it maps to, the affected resource(s), and
the framework requirement (e.g. SOC 2 CC6.1).

## 2. Triage into the fix queue

Drop each failing check into a table and classify it. Most failures fall into a small set of fix
types:

| Fix type                 | What it means                                            | Typical effort                            |
| ------------------------ | -------------------------------------------------------- | ----------------------------------------- |
| **Config change**        | Flip a setting (enforce MFA, enable secret scanning)     | Minutes–hours                             |
| **Engineering work**     | Build something (restore-test job, log sink, IAM rework) | Hours–days                                |
| **Evidence-only**        | The control is real but no artifact is filed             | Minutes                                   |
| **Policy / doc**         | Needs a written policy, not an engineering change        | Out of scope — see a lawyer / policy tool |
| **False positive / N/A** | Doesn't apply; mark as such _with a justification_       | Minutes                                   |

**The trap most teams fall into:** treating every red check as equal. A one-click "enforce MFA" and
a "build a tested DR restore process" are both one red row in the dashboard, but one is five minutes
and the other is a week. Sizing them is what makes the backlog real.

## 3. Prioritize

Work the queue in this order:

1. **Deal-blocking + config change** — fastest wins that unblock revenue. Do these today.
2. **Deal-blocking + engineering work** — scope and schedule immediately.
3. **Evidence-only** — the control already passes; you just need to export and file the artifact.
   Cheap, high ratio of dashboard-green per hour.
4. **Everything else** — the long tail, worked down on cadence.

## 4. Close the loop (don't skip this)

A platform check goes green when the underlying integration re-scans — but your _audit evidence_
still needs to live somewhere durable. For each fixed check:

1. Make the change (config or code).
2. Export the proof and file it in your [evidence folder](evidence-folder-template/README.md) under
   the right category.
3. Update the [evidence register](evidence-register.md) row to `current`.
4. Confirm the platform re-scanned and the check is green.

## Worked example

| Failing check (platform)         | Fix type         | Action                                                   | Evidence filed                                   |
| -------------------------------- | ---------------- | -------------------------------------------------------- | ------------------------------------------------ |
| "MFA not enforced for all users" | Config change    | Enforce 2SV org-wide in the IdP                          | `01-access-controls/mfa-enforcement.md`          |
| "Production branch unprotected"  | Config change    | Apply org-level ruleset requiring review                 | `03-cicd-change-management/branch-protection.md` |
| "No evidence of restore test"    | Engineering work | Build + run a quarterly restore drill, record row counts | `05-backups-recovery/restore-test.md`            |
| "Audit log retention < 365d"     | Config change    | Set storage lifecycle to ≥ 1 year + lock                 | `02-logging-monitoring/retention-policy.md`      |
| "Secret scanning disabled"       | Config change    | Enable secret scanning + push protection org-wide        | `04-vulnerability-secrets/secret-scanning.md`    |

---

The platform tells you _what's_ wrong. This queue is _how_ you fix it. If you'd rather an engineer
work the queue down and hand back merged fixes plus a populated evidence folder, that's a Controls
Review: see the [sample report](https://musabdulai.com/sample-report), then
[book a 15-minute fit call](https://musabdulai.com/call).
