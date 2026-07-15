# Evidence sharing log

The [evidence register](evidence-register.md) tracks **what evidence you have**. This log tracks
**where it went**. They are deliberately separate: one is a control inventory, the other is a
disclosure record, and collapsing them into one sheet means you can't answer either question well.

Keep this log private. It is an internal record, not something you share.

## Why you want this

Six months after a deal, someone asks: "Did we send them the raw IAM export or the sanitized one?"
Or a prospect goes quiet and you need to revoke access. Or a secret turns up in an artifact and you
need to know every recipient who might have seen it. Without a log, all three are guesswork.

It also makes the [safe evidence sharing](safe-evidence-sharing.md) discipline auditable — an expiry
you never recorded is an expiry nobody enforces.

## The log

| #   | Date shared  | Recipient (person) | Organization | Artifact                        | Classification | Approver    | Channel                | Expiry       | Revoked      | Notes                        |
| --- | ------------ | ------------------ | ------------ | ------------------------------- | -------------- | ----------- | ---------------------- | ------------ | ------------ | ---------------------------- |
| 1   | `2026-07-02` | _J. Rivera_        | _Acme Corp_  | `iam-policy-2026-Q3.buyer.json` | Sanitized      | _M. Okafor_ | Data room (named user) | `2026-08-01` | `2026-07-29` | Security review; NDA on file |
| 2   |              |                    |              |                                 | Sanitized      |             |                        |              |              |                              |
| 3   |              |                    |              |                                 |                |             |                        |              |              |                              |

### Column guide

| Column             | What goes in it                                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------------------------------- |
| **Date shared**    | `YYYY-MM-DD` the access was granted or the file sent.                                                            |
| **Recipient**      | The named individual, not "the Acme team." Access is granted to people.                                          |
| **Organization**   | Their company. Lets you answer "what does Acme have?" in one filter.                                             |
| **Artifact**       | The exact filename/version sent — not "the IAM stuff." Version matters when you re-share later.                  |
| **Classification** | `Sanitized` / `Raw original` / `Third-party attestation`. See [safe evidence sharing](safe-evidence-sharing.md). |
| **Approver**       | Who signed off. For a `Raw original` this should never be the person sending it.                                 |
| **Channel**        | Data room, scoped drive folder, trust center, email. Named-user links, not "anyone with the link."               |
| **Expiry**         | When the grant is set to lapse. Blank means indefinite — which should be rare and deliberate.                    |
| **Revoked**        | When you actually revoked it. A blank here past the expiry date is your work queue.                              |
| **Notes**          | NDA status, the deal or engagement it relates to, anything unusual.                                              |

## Copy-paste CSV

```csv
date_shared,recipient,organization,artifact,classification,approver,channel,expiry,revoked,notes
2026-07-02,J. Rivera,Acme Corp,iam-policy-2026-Q3.buyer.json,Sanitized,M. Okafor,Data room (named user),2026-08-01,2026-07-29,Security review; NDA on file
```

## Working the log

1. **Log at the moment of sharing**, not later. A log reconstructed from memory is worth very
   little.
2. **Review it on a cadence** — monthly is enough for most teams. Any row past its expiry with an
   empty `Revoked` is an action.
3. **Revoke and record.** Closing the access without writing the date back means the next reviewer
   can't tell revoked from forgotten.
4. **If a secret is ever found in a shared artifact**, filter this log by that artifact — every row
   is a recipient who may have seen it, and every one of them is a reason to rotate rather than
   hope.

---

> **Not legal advice.** What you are permitted or required to disclose — and to whom — depends on
> your contracts, NDAs, customer commitments, and regulatory obligations. This log is an operational
> record, not a compliance determination. Check with counsel where the stakes warrant it.
