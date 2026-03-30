# 🚨 Incident Response Playbook

> A structured incident response framework with detection procedures, escalation checklists, and response templates for common security incidents — aligned with **NIST SP 800-61** and **ISO/IEC 27035**.

![NIST](https://img.shields.io/badge/Aligned-NIST%20SP%20800--61-blue?style=flat)
![ISO](https://img.shields.io/badge/Aligned-ISO%2027035-green?style=flat)
![MITRE](https://img.shields.io/badge/Referenced-MITRE%20ATT%26CK-red?style=flat)

---

## Overview

This playbook provides ready-to-use incident response procedures for security teams, IT consultants, and managed service providers. Each playbook follows the NIST SP 800-61 incident lifecycle and includes detection indicators, containment steps, eradication procedures, recovery actions, and post-incident requirements.

**Incident Types Covered:**
1. 🎣 Phishing / Business Email Compromise (BEC)
2. 🔐 Ransomware
3. 📊 Data Breach / Unauthorized Data Access
4. 👤 Insider Threat
5. 🌐 Web Application Attack (SQLi, XSS, CSRF)
6. 🔑 Account Compromise / Credential Theft
7. 🕵️ Malware Infection
8. 🚫 Denial of Service (DoS/DDoS)

---

## NIST SP 800-61 Incident Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│                   INCIDENT RESPONSE LIFECYCLE                │
├──────────────┬──────────────┬──────────────┬────────────────┤
│ PREPARATION  │  DETECTION & │ CONTAINMENT, │  POST-INCIDENT │
│              │  ANALYSIS    │ ERADICATION  │  ACTIVITY      │
│ • Policies   │ • Monitoring │ & RECOVERY   │ • Lessons      │
│ • Tools      │ • Triage     │ • Isolate    │   learned      │
│ • Training   │ • Severity   │ • Remove     │ • Report       │
│ • Contacts   │   scoring    │ • Restore    │ • Improve      │
└──────────────┴──────────────┴──────────────┴────────────────┘
```

---

## Severity Classification

| Severity | Score | Description | Response SLA | Example |
|---|---|---|---|---|
| 🔴 **P1 — Critical** | 5 | Active breach, data exfiltration, ransomware | 15 min | Ransomware spreading across network |
| 🟠 **P2 — High** | 4 | Confirmed compromise, significant risk | 1 hour | Compromised admin account |
| 🟡 **P3 — Medium** | 3 | Suspected compromise, limited impact | 4 hours | Phishing email clicked, no payload |
| 🟢 **P4 — Low** | 2 | Security event, unlikely to cause harm | 24 hours | Failed login attempts (under threshold) |
| ⚪ **P5 — Informational** | 1 | No immediate risk, worth logging | 72 hours | Policy violation, no data at risk |

---

## Playbook 1 — Phishing / Business Email Compromise (BEC)

### Detection Indicators

- User reports suspicious email with unexpected link or attachment
- Email from spoofed domain similar to a trusted vendor (e.g., `micros0ft.com`)
- Unexpected wire transfer request from apparent executive
- Email security tool (Proofpoint, Defender) flagged and quarantined message
- Login from unusual geography shortly after email interaction
- MFA prompt the user did not initiate

### Immediate Actions (0–15 minutes)

- [ ] **Contain:** Quarantine the suspicious email from all mailboxes (use admin email purge)
- [ ] **Preserve:** Export email headers and body before deletion for forensics
- [ ] **Assess:** Did the user click the link or open the attachment?
- [ ] **Assess:** Did the user enter credentials on any external site?
- [ ] **Escalate:** If credentials entered → escalate to P2 (Account Compromise playbook)
- [ ] **Notify:** Alert the security team and affected user's manager

### Investigation (15–60 minutes)

- [ ] Pull email headers — check `Received`, `Reply-To`, `Return-Path` for spoofing indicators
- [ ] Check email gateway logs for how many users received the same message
- [ ] Review user's authentication logs — any successful logins in the past 24 hours from unusual locations?
- [ ] Check if any email forwarding rules were created (common BEC persistence)
- [ ] Review outbound emails from the compromised account for data exfiltration

**Key Log Sources:**
| Source | What to Look For |
|---|---|
| Email gateway (M365/Google) | Delivery, quarantine, forwarding rules |
| Azure AD / Okta sign-in logs | Logins post-phishing, MFA bypass attempts |
| Endpoint EDR | Process execution if attachment opened |
| Proxy/DNS logs | Connections to phishing domain |

### Containment

- [ ] Reset user's password immediately (do not notify attacker)
- [ ] Revoke all active sessions in Azure AD / Google Workspace
- [ ] Remove any email forwarding rules the attacker may have set
- [ ] Block the phishing domain at email gateway and DNS/proxy
- [ ] Enable enhanced monitoring on the affected account for 30 days

### Eradication & Recovery

- [ ] Verify no persistence mechanisms (inbox rules, OAuth app grants, delegate access)
- [ ] Remove any malicious OAuth application grants
- [ ] Re-enable account with new password and confirmed MFA enrollment
- [ ] Restore any emails deleted by the attacker (if applicable)

### Post-Incident

- [ ] Send phishing awareness reminder to all staff
- [ ] Review and update email security rules (DMARC, DKIM, SPF)
- [ ] Document timeline, root cause, and lessons learned
- [ ] File report per regulatory requirements if PII was accessed

---

## Playbook 2 — Ransomware

### Detection Indicators

- Files renamed with unusual extensions (`.locked`, `.encrypted`, `.crypted`)
- Ransom note files appearing in directories (`README.txt`, `DECRYPT_FILES.html`)
- Unusual process activity (mass file read/write operations)
- Antivirus / EDR alert on ransomware strain
- Lateral movement detected in network logs
- Backup deletion commands observed (`vssadmin delete shadows`)

### Immediate Actions (0–15 minutes) — **MOVE FAST**

- [ ] **ISOLATE IMMEDIATELY:** Disconnect affected system(s) from network (unplug Ethernet; disable Wi-Fi)
- [ ] **DO NOT SHUT DOWN** the machine — volatile memory may contain encryption keys
- [ ] **Preserve:** Note exact time of discovery and last known clean state
- [ ] **Alert:** Declare P1 incident — notify security lead, IT director, and management immediately
- [ ] **Assess scope:** How many systems are showing encryption? Check file servers, shared drives
- [ ] **Isolate network segments** containing affected systems at the switch/VLAN level

### Investigation (15–60 minutes)

- [ ] Identify ransomware strain — check ransom note URL against [ID Ransomware](https://id-ransomware.malwarehunterteam.com/)
- [ ] Determine patient zero — which system was first infected?
- [ ] Identify initial access vector (phishing email? RDP brute force? Vulnerable software?)
- [ ] Review SIEM/EDR for lateral movement timeline (which systems were accessed from patient zero?)
- [ ] Check backup status — are backups intact and isolated from the infected environment?

**MITRE ATT&CK Techniques to Investigate:**
| Technique | ID | What to Look For |
|---|---|---|
| Phishing | T1566 | Email logs around infection time |
| Valid Accounts | T1078 | Unusual logins before encryption |
| Remote Services | T1021 | RDP/SMB lateral movement |
| Inhibit System Recovery | T1490 | VSS deletion commands |
| Data Encrypted for Impact | T1486 | Mass file modification events |

### Containment

- [ ] Isolate all confirmed and suspected affected systems at network level
- [ ] Disable compromised accounts used for lateral movement
- [ ] Block C2 (command and control) domains/IPs at firewall
- [ ] Preserve system images before any remediation (forensic imaging)
- [ ] Determine if data exfiltration occurred before encryption (double-extortion check)

### Eradication & Recovery

- [ ] **Do not pay ransom** without legal/executive approval and law enforcement consultation
- [ ] Contact law enforcement (FBI IC3 at ic3.gov) — required for cyber insurance claims
- [ ] Check for decryptors at [NoMoreRansom.org](https://www.nomoreransom.org/)
- [ ] Wipe and rebuild affected systems from known-good images
- [ ] Restore data from clean, verified backups (test restore integrity before production use)
- [ ] Patch the initial access vector before reconnecting to network
- [ ] Reset all credentials that may have been exposed

### Post-Incident

- [ ] Conduct full forensic investigation to determine full scope
- [ ] Notify cyber insurance carrier within required timeframe
- [ ] Assess regulatory notification obligations (HIPAA 60 days, GDPR 72 hours, state laws vary)
- [ ] Implement offline/immutable backup strategy
- [ ] Conduct tabletop exercise with updated playbook within 30 days

---

## Playbook 3 — Data Breach / Unauthorized Data Access

### Detection Indicators

- Anomalous data transfer volume from internal systems to external destinations
- Database query returning unusual amounts of records (bulk export pattern)
- Alert from DLP (Data Loss Prevention) tool
- Unexpected access to sensitive files or directories outside business hours
- Third-party notification or public disclosure of stolen data
- Employee reports missing or modified records

### Immediate Actions (0–30 minutes)

- [ ] **Identify:** What data was accessed? PII, PHI, financial records, credentials?
- [ ] **Contain:** Revoke access for suspected account(s) immediately
- [ ] **Preserve:** Enable enhanced logging on affected systems; do not alter logs
- [ ] **Scope:** How many records potentially affected?
- [ ] **Escalate:** Notify legal, compliance, and executive leadership

### Investigation

- [ ] Review access logs for the compromised data store (database, S3, SharePoint)
- [ ] Identify all IP addresses and user accounts that accessed the data
- [ ] Determine if data was copied, downloaded, or transmitted externally
- [ ] Review DLP and proxy logs for outbound data transfers
- [ ] Collect forensic evidence before any remediation steps

### Notification Requirements

| Regulation | Trigger | Notification Window | Notified Party |
|---|---|---|---|
| HIPAA | PHI breach >500 individuals | 60 days | HHS + affected individuals |
| GDPR | Any personal data breach | 72 hours | Supervisory authority |
| CCPA | Unencrypted personal info | "Expedient" | Affected California residents |
| State Laws | Varies by state | 30–90 days | State AG + individuals |

> ⚠️ **Always consult legal counsel before sending breach notifications.**

---

## Playbook 4 — Account Compromise / Credential Theft

### Detection Indicators

- Login from new country/IP the user has never used
- MFA challenge the user did not initiate
- Impossible travel alert (login from NYC, then London 30 min later)
- Multiple failed MFA attempts followed by success (MFA fatigue attack)
- Unusual application access (SharePoint, HR system, financial apps)
- User reports they cannot log into their own account

### Immediate Actions (0–15 minutes)

- [ ] **Revoke all active sessions** for the account (Azure AD: Revoke Sign-in Sessions)
- [ ] **Reset password** immediately using a secure channel (not email if account is compromised)
- [ ] **Re-enroll MFA** — attacker may have registered their own authenticator device
- [ ] **Review OAuth grants** — remove any apps the attacker may have authorized
- [ ] **Check email rules** — attackers routinely add forwarding/deletion rules for persistence
- [ ] **Audit recent activity** — what did the attacker access in the compromised account?

### Investigation

- [ ] Pull full sign-in log for the account for the past 30 days
- [ ] Identify the attack vector (phishing? password spray? credential stuffing from breach?)
- [ ] Check HaveIBeenPwned or internal dark web monitoring for credential exposure
- [ ] Review all actions taken under the compromised account
- [ ] Assess if the attacker used the account to pivot to other systems

---

## General IR Contacts Template

```
INCIDENT RESPONSE CONTACTS
─────────────────────────────────────────────────────
Security Lead:          [Name]          [Phone / Email]
IT Director:            [Name]          [Phone / Email]
Legal Counsel:          [Name]          [Phone / Email]
CISO / Executive:       [Name]          [Phone / Email]
Cyber Insurance:        [Carrier]       [Policy #] [24hr Hotline]
Law Enforcement:        FBI IC3         ic3.gov / 1-800-CALL-FBI
External IR Firm:       [Firm Name]     [24hr Hotline]
─────────────────────────────────────────────────────
SIEM / Security Tools:  [Tool + Access URL]
EDR Console:            [Tool + Access URL]
Email Admin Console:    [Admin URL]
IdP Admin (Azure/Okta): [Admin URL]
─────────────────────────────────────────────────────
```

---

## Post-Incident Report Template

```markdown
## Incident Report — [Incident ID]

**Date/Time Detected:**
**Date/Time Contained:**
**Date/Time Resolved:**
**Severity:**
**Incident Type:**
**Analyst:**

### Executive Summary
[2–3 sentence summary for non-technical leadership]

### Timeline
| Time | Event |
|---|---|
| | Initial detection |
| | Escalation |
| | Containment |
| | Eradication |
| | Recovery |

### Root Cause
[What enabled this incident?]

### Impact
- Systems affected:
- Data affected (type and volume):
- Users affected:
- Business impact:

### Containment & Remediation Steps
[What was done to stop and fix it]

### Lessons Learned
[What went well / what needs improvement]

### Action Items
| Action | Owner | Due Date |
|---|---|---|
| | | |

### Regulatory Notifications Required?
☐ HIPAA  ☐ GDPR  ☐ CCPA  ☐ State Law  ☐ None
```

---

## References

- [NIST SP 800-61 Rev 2 — Computer Security Incident Handling Guide](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)
- [ISO/IEC 27035 — Information Security Incident Management](https://www.iso.org/standard/78973.html)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [CISA Incident Handling Guide](https://www.cisa.gov/topics/cyber-threats-and-advisories/incident-response)
- [NoMoreRansom — Ransomware Decryption Tools](https://www.nomoreransom.org/)
- [FBI IC3 — Cyber Crime Reporting](https://ic3.gov)

---

## Author

**Batuhan Satilmis** — Cybersecurity Analyst & IT Security Consultant
- 🌐 [forsmantech.com](https://forsmantech.com)
- 💼 [LinkedIn](https://linkedin.com/in/batuhan-satilmis)

---

## License

MIT License — adapt freely for your organization's use.

> ⚠️ This playbook is a framework, not a legal document. Always consult legal counsel for regulatory notification obligations specific to your jurisdiction and industry.
