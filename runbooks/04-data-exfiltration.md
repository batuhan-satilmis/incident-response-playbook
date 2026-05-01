# 04 — Data Exfiltration

> Evidence of unauthorized data egress. Could be insider, compromised account, or third-party post-breach. Containment is about **stopping the bleed** while **preserving evidence** so you can scope what left.

## Detection signals

- DLP alert: large transfer to personal email / consumer cloud / unrecognized domain.
- Anomalous user behavior: an account that normally exports 0-5 records exports 50,000.
- Cloud audit log: unusual `S3:GetObject` / `Storage Blob Read` / `bigquery.export` activity, especially cross-account or to public buckets.
- Outbound network spike to file-sharing providers (mega, transfer.sh, anonfiles, Discord CDN, Telegram).
- A leaving employee whose download volume accelerated in their final two weeks.
- External report (ransomware leak site, dark-web monitor, journalist) of your data appearing somewhere it shouldn't.

## First 15 minutes

1. **Contain the source** — without tipping the actor off if possible:
   - Compromised cloud account: rotate credentials, revoke active sessions, start the [01-cloud-credential-compromise](./01-cloud-credential-compromise.md) flow alongside this one.
   - Internal user (insider): disable account, revoke session, remove from groups. **Do not** confront the user yet.
   - External vendor / third party: notify them through the relationship channel; do not assume they know.
2. **Quantify the immediate scope** — what was accessed, in what window, by whom?
3. **Snapshot evidence**:
   - Cloud audit logs for the actor / time window.
   - DLP / proxy / firewall logs of the egress.
   - Mail rules + Sent Items if email-based.
   - Endpoint EDR if the path is workstation → external.
4. **Notify legal counsel and Chief Privacy Officer (or designated equivalent).** Data exfiltration is a legal-first incident — your IR steps are constrained by what the lawyers need preserved and by notification timelines that start *now*.

## Investigation

- **What was taken** — not just "documents," but: which records, which fields, whose data?
- **When did it start** — earliest known access. Often days or weeks before detection.
- **How did the actor get in** — credential phishing, leaked secret, insider abuse, third-party compromise?
- **Where did it go** — destination IPs, domains, S3 buckets, email addresses. Some of these you can subpoena / serve preservation orders on; speed matters.
- **What else did the actor touch** — assume the visible exfil isn't the only one. Map every action they took.
- **Are they still in?** — until you've definitively closed the access path, assume yes.

Reference: [MITRE ATT&CK Exfiltration tactic](https://attack.mitre.org/tactics/TA0010/).

## Eradication

1. **Close every access path** the actor used: credentials, OAuth grants, SSH keys, API tokens, federated identity, vendor access.
2. **Force token expiration** on every identity that touched the affected resources in the window. Cleanup all sessions.
3. **Revoke any third-party app consents** that look unfamiliar.
4. **Patch the access path's root cause**: phishing-resistant MFA on the involved identities; deny-list the destinations at the egress proxy; tighten DLP rules; add alerts on the specific behavior pattern.

## Recovery

- Resume normal operations only after the access path is verifiably closed and monitored.
- Apply heightened monitoring to the affected resource class for at least 90 days.
- Coordinate with comms: customer notification is a separate workstream and should not block IR work.

## Notification & legal

This is the part most teams underprepare for. The clock starts at "discovery," which is often interpreted as the first credible signal — even before full scope is known.

| Regime | Trigger | Window |
|---|---|---|
| HIPAA Breach Notification Rule | PHI exposure | 60 days to individuals; immediately if > 500 affected (HHS + media) |
| State breach laws | PII residents of that state | Varies — California: most expedient time, generally < 30 days |
| GDPR | EU/EEA personal data | 72 hours to supervisory authority |
| SEC (public companies) | Material cybersecurity incident | 4 business days from materiality determination |
| Customer contracts | Per MSA / DPA | Often 24-72 hours |
| Cyber-insurance | Per policy | Often 24-48 hours |

Get counsel in the loop in the first hour. Many of these windows can be missed inadvertently while the IR team is heads-down.

## Post-incident

- [Post-incident report](../templates/post-incident-report.md) within 72 hours, but expect the legal-facing version to take longer (it goes to outside counsel).
- Update DLP rules with the new exfiltration pattern.
- Review egress-monitoring coverage. The blind spot that allowed this is your priority improvement.
- If the access path was credential compromise: phishing-resistant MFA (WebAuthn / passkeys / hardware keys) on every identity that can read sensitive data.
- If the access path was insider: review just-in-time access models so default privilege levels are smaller.
- If the access path was third party: third-party risk-management process enhancements (continuous monitoring, breach notification clauses).

## Reporting obligations summary

- Internal: leadership, board notification triggers (per your charter).
- Regulators per the table above.
- Affected individuals per applicable laws.
- Law enforcement: FBI / Secret Service, especially if extortion is involved.
- Customers: per contract, but get counsel input on the wording.
