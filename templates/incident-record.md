# Incident Record — INC-YYYY-NNNN

> The single durable record of an incident. Fill as you go. The Scribe owns this. Don't summarize — capture timestamps, actions, and observations as they happen.

## Header

| Field | Value |
|---|---|
| Incident ID | INC-YYYY-NNNN |
| Severity | Sev1 / Sev2 / Sev3 |
| Status | Active · Contained · Resolved · Post-mortem |
| Declared at | YYYY-MM-DD HH:MM UTC |
| Declared by | <name> |
| Incident Commander | <name> |
| Investigator(s) | <name(s)> |
| Communications lead | <name> |
| Scribe | <name> |
| Channel | #inc-YYYY-NNNN |

## Initial alert / signal

Paste verbatim or screenshot. Include source (EDR / SIEM / user report / external), timestamp, and any unique identifiers.

## Affected scope (as known — update over time)

- Hosts:
- Identities:
- Data:
- Systems / services:
- Customers / tenants:

## Timeline (UTC)

> Append-only. New events go at the bottom. Don't edit prior entries — if a new fact contradicts an old one, add a new entry that updates it.

| Time | Actor | Event |
|---|---|---|
| HH:MM | system | Initial alert: <copy> |
| HH:MM | <name> | Acknowledged alert |
| HH:MM | <name> | Opened incident channel, paged IC |
| HH:MM | <name> | Isolated host <id> via EDR |
| HH:MM | <name> | Confirmed scope: <details> |
| HH:MM | <name> | Container action: <details> |
| HH:MM | <name> | Investigation finding: <details> |
| HH:MM | IC | Promoted to Sev<n> after <reason> |

## Indicators of compromise (IOCs)

| Type | Value | Source | First seen |
|---|---|---|---|
| Hash | | | |
| IP | | | |
| Domain | | | |
| User | | | |
| Process | | | |

## Decisions

> Decisions made during the incident, with rationale. Useful for the post-mortem.

| Time | Decision | Rationale | Decision-maker |
|---|---|---|---|
| HH:MM | Did not power off host XYZ | Preserve volatile memory for forensics | IC |

## Open questions

- [ ] Question 1 — owner: <name>
- [ ] Question 2 — owner: <name>

## Closeout

| Field | Value |
|---|---|
| Resolved at | YYYY-MM-DD HH:MM UTC |
| Total duration | HH:MM |
| Root cause (1 line) | |
| Customer impact (1 line) | |
| Notification triggers met | yes / no — see [post-incident report](./post-incident-report.md) |
| Post-mortem owner | <name> |
| Post-mortem due | YYYY-MM-DD |
