# 03 — Business Email Compromise (BEC)

> A user's M365 / Google Workspace mailbox is suspected to be controlled by an attacker. BEC is the highest-dollar fraud category for SMBs — wire-fraud requests originating from a CEO's "real" mailbox bypass every other control.

## Detection signals

- New inbox rule that auto-forwards or auto-deletes — especially rules that move messages to RSS Subscriptions, Notes, or Junk based on financial keywords.
- Sign-in from a new country or impossible-travel pattern.
- Sent items the user denies sending, or sent items with subjects like "Wire transfer," "Updated banking details," "Invoice payment."
- The user reports they can't see their own legitimate emails (because attacker is moving them to hide payment fraud).
- Reports from accounting that wire instructions changed.

## First 15 minutes

1. **Block sign-in for the affected user.**
   - M365: `Update-MgUser -UserId user@... -AccountEnabled:$false`
   - Or via Entra admin center: User → Account Status → Block sign-in.
2. **Revoke active sessions and refresh tokens.**
   - `Revoke-MgUserSignInSession -UserId user@...`
3. **Snapshot the mailbox state before changing anything else.** Inbox rules + sent items + mailbox audit log are evidence.
4. **Notify finance/accounting** to **stop any in-flight wires associated with that user.** Many BECs are caught here.
5. **Preserve evidence in the user's mailbox** — *do not* let the user log in to "check things." The attacker may still hold the session.

## Investigation

- **Inbox rules**: list them all.
  ```powershell
  Get-InboxRule -Mailbox user@contoso.com | Format-List Name,Enabled,Description
  ```
  Anything that moves, deletes, or forwards based on financial keywords is hostile.
- **Sent Items + Deleted Items**: messages the attacker sent will often live here briefly before they delete.
- **Mailbox audit log**:
  ```powershell
  Search-MailboxAuditLog -Identity user@contoso.com -StartDate (Get-Date).AddDays(-30) -ShowDetails
  ```
- **Sign-in log**: 
  ```powershell
  Get-MgAuditLogSignIn -Filter "userPrincipalName eq 'user@contoso.com'" -Top 200
  ```
  Look for IPs / countries the user has never used, ISP "VPN" / "Hosting Provider" patterns.
- **Application consents**: did the user grant OAuth permissions to a malicious app? `Get-MgUserOauth2PermissionGrant`.
- **MFA status**: was MFA enabled? Was an MFA method added recently (attacker provisioning)?
- **Wire-fraud check**: external email where finance was asked to update banking info. If yes → escalate to finance + legal immediately.

## Eradication

1. **Reset password** to a long random one. Do not let the user pick it yet.
2. **Re-enrol MFA from scratch.** Remove existing methods and require fresh enrolment from a known-good device.
3. **Delete every hostile inbox rule.**
4. **Revoke OAuth consents** to any app the user did not authorize.
5. **Search-and-purge** any malicious emails the attacker sent into the org from this mailbox:
   ```powershell
   New-ComplianceSearch -Name "BEC-INC-2026-0003" -ContentMatchQuery '...subject or body keywords...' \
       -ExchangeLocation All
   New-ComplianceSearchAction -SearchName "BEC-INC-2026-0003" -Purge -PurgeType SoftDelete
   ```
6. **Audit other accounts** for the same attacker pattern — BEC actors frequently lateral to several mailboxes before being caught.

## Recovery

- Re-enable the user's account only after password + MFA reset and after the user is briefed.
- Brief the user privately: do not send instructions over email; phone or in-person.
- Send a *narrow* internal advisory — "any email from <user> in the last X days asking for a wire or banking change should be ignored and reported."
- If wire fraud succeeded: file with [IC3](https://www.ic3.gov) immediately and engage your bank's fraud department within 24 hours. Time matters — funds can sometimes be recalled within ~72 hours via the [Financial Fraud Kill Chain](https://www.fbi.gov/contact-us/field-offices/atlanta/news/press-releases/business-email-compromise-the-12-billion-scam).

## Post-incident

- Tighten conditional access: country / IP allow-list for finance roles, require compliant device for high-risk operations.
- Disable legacy auth (basic auth, IMAP/POP) if not already — BEC attackers love it because it bypasses MFA.
- Train finance: every banking-detail change must be verified out-of-band on a known phone number — never the number in the email signature.
- Write the [post-incident report](../templates/post-incident-report.md). If wire fraud occurred, leadership and counsel must be in the loop.

## Reporting obligations

- If wire fraud succeeded with customer funds → contractual + cyber-insurance notifications.
- If PHI was in the mailbox → HIPAA notification process.
- If the user is regulated (lawyer, healthcare) → state-specific breach laws may apply.
