# incident-response-playbook

> NIST SP 800-61–aligned runbooks for the incident types that hit small and mid-sized organizations most often. Built for actual use during an incident, not for a binder on a shelf.

Each runbook is structured the same way:

```
1. Detection signals  — what tripped this runbook
2. First 15 minutes   — preserve, contain, notify
3. Investigation      — what to confirm, what to collect
4. Eradication        — how to remove the foothold
5. Recovery           — how to safely resume
6. Post-incident      — RCA and lessons learned
```

The **First 15 minutes** is deliberately short. When something is on fire, the responder doesn't need a 40-page manual; they need 5 actions in the right order.

## Available runbooks

| Runbook | When to open |
|---|---|
| [00-master-checklist.md](./runbooks/00-master-checklist.md) | Anything. Always start here. |
| [01-cloud-credential-compromise.md](./runbooks/01-cloud-credential-compromise.md) | Suspected leak / theft of AWS, Azure, or GCP credentials. |
| [02-ransomware-endpoint.md](./runbooks/02-ransomware-endpoint.md) | Ransomware detected on a workstation or server. |
| [03-business-email-compromise.md](./runbooks/03-business-email-compromise.md) | A user's M365 / Google Workspace mailbox is suspected to be controlled by an attacker. |
| [04-data-exfiltration.md](./runbooks/04-data-exfiltration.md) | Evidence of unauthorized data egress. |

## Templates

| Template | When to use |
|---|---|
| [executive-summary.md](./templates/executive-summary.md) | First written communication to leadership during an incident. |
| [incident-record.md](./templates/incident-record.md) | The single durable record of an incident. Fill as you go. |
| [post-incident-report.md](./templates/post-incident-report.md) | Written within 72 hours of incident close. |

## Roles

This playbook assumes a small org (no dedicated SOC). It expects:

- **Incident Commander (IC)**: runs the response. Almost never the same person investigating.
- **Investigator**: hands-on with logs, hosts, identities.
- **Communications lead**: writes the executive summary and customer-facing comms.
- **Scribe**: maintains the incident record.

In a 5-person company, one person can wear two hats — but never IC + Investigator. The IC keeps the timeline; the Investigator keeps their head down.

## Standards alignment

| Standard | Where it shows up |
|---|---|
| NIST SP 800-61 (Computer Security Incident Handling Guide) | Phase structure (preparation, detection, containment, eradication, recovery, post-incident). |
| NIST CSF | Detect (DE.AE), Respond (RS), Recover (RC) families. |
| HIPAA Security Rule | Breach notification timelines and content where PHI is involved. |
| SOC 2 CC7.3 | Incident response process documented and tested. |

## License

MIT
