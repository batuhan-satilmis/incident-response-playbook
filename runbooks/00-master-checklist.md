# Master Checklist — open this first

> The next 15 minutes determine the next 15 days. Do these in order. Don't multitask.

## 0–2 min: confirm and capture

- [ ] **Read the alert / report twice.** Don't act on a partial signal.
- [ ] **Open the [Incident Record](../templates/incident-record.md) document.** Assign a unique ID like `INC-2026-0001`. Note the time you saw the alert (UTC).
- [ ] **Take a screenshot** of whatever made you suspect an incident. Save it in the incident folder.

## 2–5 min: assemble

- [ ] **Page the on-call IC.** If that's you, page someone else as IC — you're now the Investigator.
- [ ] **Open a private channel** for the incident (e.g. `#inc-2026-0001`). Slack, Teams, Discord — whatever you use.
- [ ] **Bring in the relevant subject-matter expert** (cloud admin, app engineer, M365 admin) — but not more than 3 people total at first.

## 5–10 min: classify

Based on the suspected category, open the matching runbook:

| Suspect | Runbook |
|---|---|
| Cloud credentials in the wild, mass IAM activity, anomalous compute spend | [01-cloud-credential-compromise.md](./01-cloud-credential-compromise.md) |
| File encryption / ransom note / EDR ransomware alert | [02-ransomware-endpoint.md](./02-ransomware-endpoint.md) |
| Mailbox rules created without user, unusual sent items, finance impersonation | [03-business-email-compromise.md](./03-business-email-compromise.md) |
| Unusual data egress, large queries, unfamiliar exporters | [04-data-exfiltration.md](./04-data-exfiltration.md) |

If two might apply, pick one and note it in the incident record. Don't waste time debating.

## 10–15 min: contain *minimum viable*

Following the runbook's First-15-minutes section, make the **smallest containment move that stops more damage** — not the largest cleanup. Examples:

- Disable the suspect IAM access keys (don't yet rotate every key).
- Quarantine the suspected host (don't yet reimage every workstation).
- Lock the suspect mailbox (don't yet password-reset every user).

**Aggressive over-containment turns a 1-hour incident into a 24-hour outage.** Match the response to the evidence.

## Do NOT, in the first 15 minutes

- ❌ Email all-hands. (Use the private channel until comms is approved.)
- ❌ Tweet. (Ever, during an active incident, without comms approval.)
- ❌ Delete any logs or evidence "to be safe."
- ❌ Pay a ransom. (Comes later, with leadership and counsel.)
- ❌ Power off compromised hosts. (Memory contents disappear; IR teams want them online but isolated.)

## After 15 minutes

Continue with the specific runbook's investigation and eradication steps. Update the incident record continuously. The Communications lead should be drafting the [Executive Summary](../templates/executive-summary.md) for leadership by minute 30.
