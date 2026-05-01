# 02 — Ransomware on an Endpoint

> A workstation or server is encrypting files / displaying a ransom note / detected by EDR. Containment must be **immediate but careful** — pulling the plug destroys volatile evidence; doing nothing destroys the rest of your fleet.

## Detection signals

- EDR / antivirus alert with category "ransomware" or names a known family (LockBit, BlackCat/ALPHV, Akira, Royal, Play, etc.).
- User reports files renamed with a strange extension (`.lockbit`, `.akira`, `.encrypted`) or unable to open documents.
- A `README.txt` / `readme_for_decrypt.txt` / similar appears across many directories.
- Unusual SMB / RDP activity from a single host to many others.
- Spike in disk I/O and CPU on a host outside business hours.
- Backup jobs failing — attackers commonly disable backups before the encryption phase.

## First 15 minutes

1. **Isolate the host from the network — do NOT power it off.**
   - EDR isolation feature (preferred): one-click in CrowdStrike, Defender for Endpoint, SentinelOne.
   - Failing that, disconnect the network cable / disable Wi-Fi at the host level.
   - **Why not power off**: shutting down loses memory contents (decryption keys, attacker tooling) and may trigger ransomware finalization that completes encryption.
2. **Identify scope: is this one host or many?**
   - Pull a list of hosts that have communicated with the suspect host in the last 24h.
   - Check EDR for the same indicator on other hosts.
3. **Assemble the bridge** with IC, sysadmin, and someone with EDR / SIEM access.
4. **Snapshot evidence**: from the EDR console, export the process tree, the file-write events, the network connections from the suspect host. Do **not** open files on the affected host while it's still running encryption.
5. **Do not pay** in this window. Don't even discuss it. That's a leadership + counsel + cyber-insurance conversation, with a clearer head.

## Investigation

- **Root cause / initial access**:
  - Check email gateway for malicious attachments / links delivered to the user in the last 30 days.
  - Check VPN / RDP logs for external authentication.
  - Check whether known [CVEs](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) for any internet-facing service are unpatched.
- **Lateral movement**:
  - Scan EDR for the same hash / behavior on other hosts.
  - Pull authentication logs for any privileged account that touched the host in the last 7 days.
  - Look for SMB scanning / WMI / PsExec / RDP from the affected host outward.
- **Persistence**:
  - Scheduled tasks created in the last 30 days.
  - New services / drivers.
  - WMI event subscriptions.
  - Run keys, startup folders.
  - Any new local admin accounts.
- **Backup status**: were backups deleted, encrypted, or modified? Restoring from a tampered backup re-introduces the ransomware.
- **Data theft check**: most modern ransomware operators **exfiltrate before they encrypt** — "double extortion." Look for unusual outbound traffic in the days before encryption (large transfers to file-sharing sites, cloud storage providers, attacker-controlled servers).

Reference: [MITRE ATT&CK T1486 — Data Encrypted for Impact](https://attack.mitre.org/techniques/T1486/), [CISA #StopRansomware](https://www.cisa.gov/stopransomware).

## Eradication

1. **Reimage** the affected host. Do not "clean" — modern ransomware drops persistence; assume nothing is trustworthy on disk.
2. **Reset credentials**:
   - Local admin password on the host (now wiped, but rotate the model used elsewhere).
   - Domain account that was logged in — disable, rotate password, revoke Kerberos tickets (`Reset-ADUser` + `klist purge`).
   - Any service account that authenticated to the host.
3. **Patch the initial access vector** identified in investigation.
4. **Scan adjacent hosts** with EDR for the same indicators. Reimage any positive matches.
5. **Rotate any secret material** that lived on the host (SSH keys, browser-saved passwords, API tokens in env files, Slack/Teams cookies).

## Recovery

- Restore from a **known-good backup** taken before the attack timeline. Verify the restore point is clean.
- Reintroduce the host to the network only after EDR confirms clean state.
- Watch the affected user / host for re-infection signals for 30 days.
- Reauthenticate the user from a known-good device.

## Decision: pay or not pay

This is a **leadership decision**, not an IR decision. The factors:

| Factor | Toward paying | Against paying |
|---|---|---|
| Backups exist and are clean | | ✓ — recover, don't pay |
| Operations down causing imminent material loss | ✓ | |
| Data exfiltrated and threatened to be leaked | ✓ (sometimes) | The decryption key doesn't undo the leak |
| Sanctions exposure (OFAC-listed group) | | Paying may be **illegal** |
| Cyber-insurance covers payment | ✓ | |
| Track record of the group honoring decryption | ✓ if good | Many groups don't deliver, or partial |

If pay is on the table: cyber-insurance, outside counsel, and FBI/Secret Service must be in the room. Don't negotiate solo.

## Post-incident

- [Post-incident report](../templates/post-incident-report.md) within 72 hours.
- Tabletop exercise within 30 days using this incident as the scenario.
- Backup hardening: immutable / air-gapped backups, alarmed deletion attempts.
- EDR coverage gap analysis — if any host wasn't enrolled, that's the priority fix.
- Prioritize MFA on every remote-access surface (RDP, VPN, VDI). Most ransomware initial access is brute-forced or phished credentials on un-MFA'd remote access.

## Reporting obligations

- **CISA**: incidents involving critical infrastructure should be reported within 72 hours under [CIRCIA](https://www.cisa.gov/topics/cyber-threats-and-advisories/information-sharing/cyber-incident-reporting-critical-infrastructure-act-2022-circia).
- **FBI IC3**: file at [ic3.gov](https://www.ic3.gov).
- **State breach laws**: varies by state, generally 30-90 day notice if PII / PHI was exposed.
- **HIPAA**: 60-day notification window if PHI is involved.
- **Cyber-insurance**: notify per policy terms — usually within 24-72 hours.
