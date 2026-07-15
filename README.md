# Cloud Controls Evidence Kit

A drop-in scaffold for B2B SaaS and AI-product teams to organize the engineering controls and
evidence that customer security reviews, SOC 2 readiness work, and compliance platforms (Vanta,
Drata, Secureframe) ask for. Markdown source, MIT licensed, free to fork, edit, and use.

## What's inside

| Folder / file                                                      | What it is                                                                                                                                                    |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `controls-map.md`                                                  | Master mapping: control → required evidence → where to find it → owner → update cadence. ~30 controls across 6 categories.                                    |
| `evidence-register.md`                                             | The index of your whole program: one row per control → artifact → owner → last refreshed → status. Includes a copy-paste CSV.                                 |
| `security-review-intake.md`                                        | One-page intake to scope a review before you start: trigger, deadline, framework, systems, buyer asks, evidence you already have.                             |
| `vanta-drata-secureframe-gap-export.md`                            | How to turn a compliance platform's failing-check list into a prioritized engineering fix queue.                                                              |
| `safe-evidence-sharing.md`                                         | What evidence is safe to share with a buyer vs. auditor-only or internal — redaction, secure delivery, expiry/revocation, and NDA/trust-center handling.      |
| `evidence-sharing-log.md`                                          | The disclosure log: one row per artifact sent outside the company — recipient, classification, approver, channel, expiry, revocation. Includes a CSV.         |
| `questionnaire-answer-examples.md`                                 | 12 template answers for the questions that come up on customer security questionnaires (MFA, audit logs, backups, etc.).                                      |
| `evidence-folder-template/`                                        | Ready-to-use folder layout — six numbered category folders, each with a README and 3–4 evidence templates. Drop into your evidence repo and start filling in. |
| `evidence-folder-template/_filled-example/`                        | A worked example: five control files filled in for a fictional company, so you can see what "done" looks like next to the blank templates.                    |
| `platforms/aws.md`, `gcp.md`, `azure.md`, `github.md`, `gitlab.md` | Where to find each piece of evidence on each platform: console paths, CLI commands, exports that satisfy auditors.                                            |
| `LICENSE`                                                          | MIT. Use it, edit it, ship it, redistribute it.                                                                                                               |

## How to use it

1. **Fork or download** this kit (link on the
   [evidence kit page on musabdulai.com](https://musabdulai.com/evidence-kit)).
2. **Drop `evidence-folder-template/` into your own private repo** and rename it to whatever your
   team calls evidence. Many teams use `compliance/` or `audit/`.
3. **Open `controls-map.md`** and adapt the table to your actual systems. Delete rows that don't
   apply, add rows for systems we don't cover.
4. **Fill in each template** as you collect the evidence. Each template tells you what goes there,
   where to find it, and how often to refresh it.
5. **When a customer questionnaire arrives**, paste answers from `questionnaire-answer-examples.md`
   and attach the matching evidence from your folder.

## Tone and scope

This kit is opinionated and concrete. It names specific tools (CloudTrail, Vanta, GitHub branch
protection, Vertex AI) instead of staying framework-agnostic — because that's how the controls
actually map at most SaaS shops.

It does **not** cover:

- Drafting policies (privacy policy, security policy text) — talk to a lawyer; tools like StrongDM's
  [Comply](https://github.com/strongdm/comply) are good for policy drafting.
- Compliance-platform-specific configuration (each platform has its own docs).
- Threat modeling, red-team plans, or incident response procedures — different scope.

This kit covers **the operational evidence engineering teams have to produce** when a buyer or
auditor asks for proof that a control is real.

## Disclaimer

This is a **starter evidence scaffold, not audit, legal, or attestation advice.** It does not make
you SOC 2 compliant and it is not a substitute for an auditor or a CPA firm. The control IDs,
framework cross-walks (SOC 2, ISO 27001, etc.), and platform commands are starting points — cloud
provider APIs, console paths, and audit expectations change over time. **Verify the final control
mappings and evidence with your auditor and confirm every command against the current vendor
documentation** before relying on it for a real review. The filled example uses a fictional company;
its names, dates, and numbers are illustrative.

## Updating cadence

Customer security reviews and auditors look for evidence that is **fresh**. The rule of thumb on
cadence:

| Evidence type                                                        | Refresh                         |
| -------------------------------------------------------------------- | ------------------------------- |
| Policy / config snapshots (MFA policy, branch protection, retention) | Quarterly                       |
| Operational logs / exports (IAM key inventory, deploy history)       | Quarterly                       |
| Restore tests, DR drills                                             | Quarterly or semi-annually      |
| Incident runbook, on-call rotation                                   | Annually unless a real incident |
| Vendor reviews, third-party access                                   | Annually                        |

Each evidence template includes a recommended cadence.

## License

[MIT](https://github.com/musabdulai-io/cloud-controls-evidence-kit/blob/main/LICENSE). Use it
however helps you ship.

## Maintained by

Musah Abdulai · cloud controls implementation for B2B SaaS and AI-product teams ·
[musabdulai.com](https://musabdulai.com) · hello@musabdulai.com

If this kit saved you a day of evidence-gathering work and you'd like an engineer to actually do the
work for you, the website has a sample report showing what a Controls Review deliverable looks like:
[musabdulai.com/sample-report](https://musabdulai.com/sample-report).
