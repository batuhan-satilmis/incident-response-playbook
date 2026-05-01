# 01 — Cloud Credential Compromise

> AWS / Azure / GCP key or session is suspected to be in attacker hands. The clock is short — credentials are usually monetized within hours.

## Detection signals

- AWS Access Key listed in a public GitHub commit / paste site / leaked secrets database (GitGuardian, TruffleHog, AWS abuse notice).
- CloudTrail shows API calls from unfamiliar IPs / geographies / user-agents.
- Sudden GPU / large EC2 instance launch in regions you don't use.
- IAM activity from a long-dormant key.
- Unsigned support emails from cloud provider asking about specific charges.

## First 15 minutes

1. **Identify the credential.** Access key? Session token? IAM role assumption? Federated user? Different responses follow.
2. **Disable the key.**
   - AWS: `aws iam update-access-key --access-key-id AKIA... --status Inactive --user-name <user>` (do not delete yet — preserve for forensics).
   - Azure: revoke the secret on the App Registration / disable the user.
3. **Identify everything that key could touch.** Pull the IAM policies attached. List affected services and resources.
4. **Snapshot evidence.**
   - AWS: enable CloudTrail data events if not already. Pull the last 24h of events for the credential.
   - Azure: pull Activity Log + Sign-in Logs for the principal.
5. **Open a private channel and notify IC.**

Do **not** rotate every key in the account in this window. Targeted disable, then investigate.

## Investigation

- **Scope**: timeline of every action the credential performed since the suspected leak time.
- **Lateral movement**: did the attacker assume any roles? Create new keys? Modify IAM policies? Add their email to a notification list?
- **Persistence checks** (these are the actions attackers take to keep access):
  - New IAM users created.
  - New access keys on existing users.
  - New trust relationships added to roles.
  - New OIDC providers / SAML providers.
  - New Lambda functions, especially with `AdministratorAccess`.
  - New EventBridge rules that re-create deleted things.
  - New CloudTrail trails routing to attacker buckets, or existing trails disabled.
  - New SSH keys / EC2 user-data scripts that re-spawn instances.
- **Data access**: any S3 GetObject on sensitive buckets? Any RDS snapshots created and shared cross-account?
- **Cost signals**: anomalous bills indicating crypto-mining or data-egress.

Reference: [MITRE ATT&CK T1078.004 — Cloud Accounts](https://attack.mitre.org/techniques/T1078/004/).

## Eradication

Once scope is clear:

1. **Rotate the credential** (delete or rotate the access key after evidence is preserved).
2. **Remove every persistence artifact** found in the investigation. Document each one in the incident record.
3. **Force token expiration.** AWS: revoke active sessions for the user. Azure: revoke refresh tokens (`Revoke-AzureADUserAllRefreshToken`).
4. **If the credential had broad access, rotate adjacent secrets** that may have been exposed in resources the attacker could read.

## Recovery

- Re-enable the credential only if needed and demonstrably clean. More commonly, replace it with a short-lived role + SSO.
- Verify production health. Cloud-credential incidents sometimes cause downstream service outages from the eradication itself.
- Re-enable any controls disabled during containment.

## Post-incident

- Run the [post-incident report](../templates/post-incident-report.md) within 72 hours.
- Pre-mortem actions:
  - **Why was the credential long-lived?** Move to short-lived (SSO, instance roles, OIDC).
  - **Where did it leak?** Source-control scan, secret-scanning in CI, employee laptop, ex-employee.
  - **Why didn't we detect it earlier?** Add a rule for first-use of any IAM key from a new region.

## Cloud-specific quick-reference

### AWS

```bash
# disable a key
aws iam update-access-key --access-key-id AKIA... --status Inactive --user-name jdoe

# revoke active sessions for a user
aws iam put-user-policy --user-name jdoe --policy-name DenyAll-temp --policy-document file://deny-all.json

# pull events for the key
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=AKIA...

# list everything the user is attached to
aws iam list-attached-user-policies --user-name jdoe
aws iam list-groups-for-user --user-name jdoe
```

### Azure / Entra ID

```powershell
# block sign-in
Update-MgUser -UserId user@contoso.com -AccountEnabled:$false

# revoke all sessions
Revoke-MgUserSignInSession -UserId user@contoso.com

# pull sign-in logs
Get-MgAuditLogSignIn -Filter "userPrincipalName eq 'user@contoso.com'" -Top 200
```

## Reporting obligations

- If customer data was accessed: review your contractual notification windows (often 24-72h).
- If PHI was accessed: HIPAA breach notification process (within 60 days).
- If the credential belonged to a paying customer's environment under a managed agreement: trigger your customer-incident process.
