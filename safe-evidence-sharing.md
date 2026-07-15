# Safe evidence sharing

You did the work: MFA is enforced, the audit logs are on, the branch is protected, and you have the
exports to prove it. Now a buyer's security team wants to see them. This is the step where teams
quietly leak account IDs, employee emails, internal IPs, and sometimes secrets — because the fastest
way to answer "send us proof" is to forward the raw export, and the raw export was never written to
leave the building.

This guide covers what's safe to share, what isn't, how to sanitize the difference, and how to
deliver and expire it. The kit tells you how to _collect_ evidence; this is how to _disclose_ it
without creating a second problem.

## The core rule: two copies of every artifact

Keep an **internal original** and a **sanitized external copy** of every piece of evidence. **Never
send the raw original to a buyer or prospect.**

- **Internal original** — the raw export, with real identifiers, stored in your private evidence
  folder (see [evidence-folder-template](evidence-folder-template/README.md)). This is what you diff
  against next quarter.
- **Sanitized external copy** — a redacted version built specifically for outside eyes. It proves
  the control is real without exposing anything an attacker could use. This is what goes to buyers.

**Auditors are a different case, and it's worth being precise about it.** An auditor under an
engagement letter usually needs the raw original — sampling redacted evidence defeats the point, and
they are contractually bound to confidentiality in a way a prospect is not. So the boundary isn't
"originals never leave," it's:

| Recipient                      | Gets                                   | Basis                                          |
| ------------------------------ | -------------------------------------- | ---------------------------------------------- |
| Buyer / prospect               | Sanitized copy only                    | No engagement; least disclosure that proves it |
| Auditor                        | Raw original, via a controlled channel | Engagement letter + confidentiality terms      |
| Internal reviewers / new hires | Raw original, scoped to their role     | Need-to-know within the company                |

Give auditors the original through the same controlled, logged, time-boxed channel described below —
"the auditor can have it" is not the same as "email it as an attachment."

Name the pair so the relationship is obvious, e.g. `iam-policy-2026-Q3.json` (internal) and
`iam-policy-2026-Q3.buyer.json` (sanitized). The sanitized copy is _derived_ from the original, so
the fact stays true — you're not fabricating, you're withholding identifiers.

## What's usually safe to share with a buyer

A buyer's security reviewer needs to believe the control exists and works. That almost never
requires raw data. Safe, sanitized artifacts:

- **Configuration-as-code** with identifiers replaced by placeholders — the Terraform, ruleset, or
  policy that _defines_ the control (project IDs → `PROJECT_ID`, emails → `user@…`).
- **The finding → fix → verification → evidence map** — a table showing each control, how it's
  implemented, and how it was verified. This is often all a buyer actually needs.
- **A statement of a setting** — "Audit Data Access logging is enabled; log retention is 400 days" —
  rather than the logs themselves.
- **Redacted verification transcripts** — the _fact_ of a rejected direct push or a passing check,
  with hostnames and users scrubbed.
- **Third-party attestations** — your SOC 2 report (usually under NDA), a pen-test summary letter,
  cloud-provider compliance certificates.
- **Policy documents** written for external readers.

## What stays auditor-only or internal

Some evidence is fine for an auditor under engagement but should not go to a prospect, and some
should never leave at all:

- **Raw IAM / access exports** with real member emails and numeric account/project numbers.
- **Actual log entries** — they carry principal emails, caller IP addresses, resource names, and
  request parameters.
- **Network diagrams, hostnames, internal IP ranges, and infrastructure topology.**
- **Vulnerability scan output and pen-test details** — the full findings are a roadmap for an
  attacker. Share the summary letter, not the raw report.
- **Break-glass / emergency-access account details** and rotation procedures.
- **Anything naming another customer** — one prospect's evidence pack must never contain another
  buyer's name, logo, or data.
- **Secrets of any kind** — see below; these are never "redact and send," they're "rotate."

## Redaction checklist

Before any artifact leaves, scrub:

- [ ] **Cloud account identifiers** — AWS account IDs, GCP project IDs/numbers, Azure subscription
      and tenant IDs → replace with `ACCOUNT_ID` / `PROJECT_ID`.
- [ ] **Employee names and emails** → `engineer@company.com` or a role (`CI service account`).
- [ ] **IP addresses and hostnames** — internal ranges, load-balancer IPs, private DNS names.
- [ ] **Customer data** — any real customer name, record, identifier, or sample row.
- [ ] **Secrets** — API keys, tokens, connection strings, private keys, passwords. If a secret ever
      appeared in an artifact, **rotate it** — redaction is not enough, because the file existed
      with it in the clear and may already be in a screenshot, log, or someone's inbox.
- [ ] **Resource ARNs / URIs** that embed account IDs or internal names.
- [ ] **Metadata** — check screenshot EXIF, PDF author/producer fields, and document properties;
      they leak usernames, machine names, and file paths.

Two practical warnings:

- **Redact from data, not with a black box.** A black rectangle over text in a PDF or image often
  leaves the text selectable underneath. Rebuild the artifact from redacted source, or flatten to an
  image _after_ deleting the text — don't just cover it.
- **Screenshots leak the frame.** Browser tabs, bookmarks, other windows, and notification banners
  routinely appear in "just a quick screenshot." Prefer exported config over screenshots; when a
  screenshot is unavoidable, crop hard.

## Secure delivery

How the sanitized copy travels matters as much as what's in it:

- **Prefer a controlled channel over email attachments.** A link to a permissioned location (a
  shared drive folder scoped to the buyer's reviewer, or a trust-center / data room) beats a file
  that lives forever in an inbox and forwards itself.
- **Scope access to named people**, not "anyone with the link," and not your whole domain.
- **Set an expiry** on the link or grant (see below).
- **Log who you gave what, and when** — one row per disclosure in the
  [evidence sharing log](evidence-sharing-log.md). Keep that separate from your
  [evidence register](evidence-register.md): the register is an inventory of the evidence you hold,
  the log is a record of where it went. You want to answer "what exactly did this buyer receive?"
  months later without guessing.
- **Avoid pasting evidence into shared Slack/Teams channels or ticketing tools** where it's
  retained, searchable, and visible beyond the intended reader.

## Expiry and access revocation

Evidence access should be temporary by default:

- **Time-box the grant.** Give the buyer's reviewer access for the length of the review (e.g. 30
  days), not indefinitely. Record the expiry in the [evidence sharing log](evidence-sharing-log.md)
  — an expiry nobody wrote down is an expiry nobody enforces.
- **Revoke when the review closes.** Put a reminder on it; access that outlives its purpose is pure
  downside.
- **Rotate anything exposed.** If a secret was ever in a shared artifact, rotate it regardless of
  whether you think it was seen.
- **Re-sanitize each time.** Don't reuse last quarter's buyer copy for a new prospect without a
  fresh scrub — details drift, and the old pack may name the wrong company.

## NDA and trust-center considerations

- **Gate sensitive artifacts behind an NDA.** SOC 2 reports, pen-test summaries, and detailed
  architecture should go out under a mutual or one-way NDA. "Send us your SOC 2" is a reasonable
  ask; "send it with no agreement" is not.
- **A trust center scales this.** A hosted trust page (Vanta, Drata, SafeBase, Secureframe, or a
  simple gated page) lets you publish the low-sensitivity artifacts openly and put the rest behind a
  click-through NDA and access request — so you answer most questionnaires without a bespoke email
  each time.
- **Match the artifact to the ask.** Most questionnaire questions are satisfied by a sanitized
  statement plus config. Reserve the NDA-gated originals for buyers who genuinely need to go deeper.

---

> **Disclaimer — this is operational guidance, not legal advice.** What you may share, what you must
> share, and what you must not share vary by contract and context: your customer agreements, NDAs
> and their survival terms, auditor engagement letters, insurance and regulatory obligations (GDPR,
> HIPAA, PCI DSS, sector rules), and any commitments in your own trust center or DPAs. Some regimes
> require disclosures this guide would tell you to withhold; some contracts forbid disclosures this
> guide treats as routine. The buyer/auditor/internal boundary described here is a sensible default,
> not a determination for your situation — confirm the sensitive calls with counsel and with whoever
> owns your customer contracts.

The short version: prove the control, not the environment. Keep the raw original internal, release
it to an auditor only through a controlled channel under engagement terms, build a sanitized copy
for buyers, redact identifiers and rotate secrets, deliver through a scoped and expiring channel,
and log every disclosure in the [evidence sharing log](evidence-sharing-log.md). If you'd rather an
engineer produce the buyer-ready pack alongside the fixes — sanitized config, verification
transcripts, and the mapping table — that's part of a Controls Review: see the
[sample report](https://musabdulai.com/sample-report), then
[book a 15-minute fit call](https://musabdulai.com/call).
