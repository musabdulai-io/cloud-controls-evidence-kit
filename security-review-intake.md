# Security review intake

Fill this out _before_ you start gathering evidence. It turns "a customer is asking for security
stuff" into a scoped, sized piece of work — and it's the same information an engineer (or a
[Controls Review](https://musabdulai.com/sample-report)) needs to help you.

Copy this file into your evidence repo and answer inline.

## 1. The trigger

- **What kicked this off?** (customer security questionnaire / auditor PBC list / investor DD /
  compliance-platform onboarding / proactive readiness)
- **Who is the requester?** (company + role of the person asking)
- **Hard deadline:** `YYYY-MM-DD` — and what happens if you miss it (deal slips, renewal blocked,
  etc.)
- **Soft target:** when you'd _like_ this done

## 2. The framework / format

- **What standard is the request based on?** (SOC 2 / ISO 27001 / CAIQ / CSA CCM / a bespoke
  questionnaire / "just send us your security docs")
- **SOC 2 specifically:** Type 1 or Type 2? (see
  [Type 1 vs Type 2](https://musabdulai.com/resources/soc2-type-1-vs-type-2-customer-deadline))
- **Observation window** (Type 2 only): start and end dates the auditor will sample
- **Compliance platform in use:** (Vanta / Drata / Secureframe / none / other)

## 3. Scope — systems in play

Check all that apply and name the specific accounts/orgs:

- [ ] **Cloud:** AWS `____` · GCP `____` · Azure `____` (account/project IDs)
- [ ] **Source control / CI:** GitHub org `____` · GitLab group `____` · CI system `____`
- [ ] **Data stores:** managed DBs `____` · object storage `____` · data warehouse `____`
- [ ] **Identity:** Okta / Google Workspace / Entra / other `____`
- [ ] **AI workloads:** model providers `____` · vector stores `____` · agent/tool surfaces `____`
- [ ] **Other production systems** that hold customer data `____`

## 4. What the buyer/auditor actually asked for

List the specific questions or PBC line items. Map each to a control category so you can answer from
your evidence folder:

| #   | Their question (verbatim)                   | Control category   | Evidence we have?  |
| --- | ------------------------------------------- | ------------------ | ------------------ |
| 1   | _e.g. "Is MFA enforced for all employees?"_ | 1. Access controls | yes / no / partial |
| 2   |                                             |                    |                    |
| 3   |                                             |                    |                    |

## 5. Evidence exports available today

Be honest — this is what sizes the work:

- [ ] IAM / role exports (per cloud)
- [ ] MFA enforcement export from the IdP
- [ ] Audit-log configuration + a sample query result
- [ ] Branch-protection / merge-approval config export
- [ ] Deploy history (last ~30 production deploys with approver)
- [ ] Dependency / secret / image scanner config + a remediation example
- [ ] Backup configuration + a real restore-test report
- [ ] Incident runbook + most recent postmortem
- [ ] AI workload controls (endpoint auth, logging, spend caps) — if applicable

## 6. Access + constraints

- **Can you grant safe read-only access** to the systems above? (yes / no / with conditions)
- **Who owns each system** and can pull exports? (names)
- **Anything off-limits** (regulated data, customer environments you can't touch)?

## 7. Output

- **What does "done" look like?** (a returned questionnaire / a populated evidence folder / a
  readiness gap list / merged fixes + evidence)
- **Who signs off** on the answers before they go back to the requester?

---

Once this page is filled in, the gaps in sections 4 and 5 are your work list. If you want an
engineer to walk the stack and turn the `no`/`partial` rows into shipped fixes + filed evidence,
that's a Controls Review: see the [sample report](https://musabdulai.com/sample-report), then
[book a 15-minute fit call](https://musabdulai.com/call).
