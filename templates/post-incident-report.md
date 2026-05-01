# Post-Incident Report — INC-YYYY-NNNN

> Written within **72 hours** of incident close. Blameless. The goal is *systemic* improvement, not personal accountability.

## Summary

| Field | Value |
|---|---|
| Incident ID | INC-YYYY-NNNN |
| Severity | Sev1 / Sev2 / Sev3 |
| Detected | YYYY-MM-DD HH:MM UTC |
| Contained | YYYY-MM-DD HH:MM UTC |
| Resolved | YYYY-MM-DD HH:MM UTC |
| Author | <name> |
| Reviewers | <names> |

**One-sentence summary**: <what happened, in plain English>

**Customer impact**: <records / accounts / systems affected, in user-meaningful terms>

## Timeline (condensed)

| Time | Event |
|---|---|
| | Initial signal |
| | Detection / declaration |
| | First containment action |
| | Eradication complete |
| | Service restored |
| | Public communication (if any) |

For the full minute-by-minute timeline, see [the incident record](./incident-record.md).

## Root cause

**Proximate cause** (the thing that triggered the alert):

**Underlying cause** (the systemic gap that allowed the proximate cause to land):

Use the **Five Whys** discipline: keep asking "why was that possible?" until you hit a systemic answer (not a person's name).

## What went well

- Things to keep doing.

## What went poorly

- Be specific. "Detection took too long" is less useful than "detection took 47 minutes because the alert routed to a Slack channel nobody monitors at night."
- Process gaps, tool gaps, knowledge gaps, communication gaps.
- This section being short usually means the team is being too generous to itself.

## Lucky breaks

- Things that helped that we don't control. We need controls we *do* own; this section names where we relied on luck.

## Action items

| # | Action | Owner | Due | Class |
|---|---|---|---|---|
| 1 | | | YYYY-MM-DD | Prevent / Detect / Respond / Process |

Action items have owners and due dates. Items without an owner *will not happen*. Items without a due date *will not happen*.

Class definitions:
- **Prevent** — closes the gap that allowed the incident.
- **Detect** — would have surfaced the incident sooner.
- **Respond** — would have made response faster or more effective.
- **Process** — fixes a process flaw revealed by the incident.

## Customer / regulatory communications sent

| Date | Audience | Channel | Content reference |
|---|---|---|---|
| | | | |

## Notification obligations (per [04-data-exfiltration.md table](../runbooks/04-data-exfiltration.md#notification--legal))

- [ ] Internal leadership / board (if triggered)
- [ ] Cyber-insurance carrier (within policy window)
- [ ] Regulators (per applicable regime)
- [ ] Affected individuals (per applicable law)
- [ ] Law enforcement (where appropriate)
- [ ] Customers (per contract)

## Detection signals — what we should add

> If we'd had alerts on the right signals, this would have been detected sooner. List the alerts we should add, with thresholds.

- Alert: `<name>` — fires on `<condition>` — routes to `<destination>`.

## Lessons (1-line each)

- The kind of takeaway you'd put on a one-pager for the next responder.

## Distribution

- Engineering / IT
- Leadership
- (If customer-facing): customer-facing summary as a separate doc

---

*Blameless, accurate, action-oriented. If a name appears in this document, it appears next to a contribution, not a cause.*
