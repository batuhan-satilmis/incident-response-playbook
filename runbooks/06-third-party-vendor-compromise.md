# 06 — Third-Party / Vendor Compromise

> Your SaaS vendor, upstream dependency, or managed provider has been breached and your data or access is potentially affected. You're a downstream victim: containment is mostly about **your integrations** with them, and the primary evidence source is **outside your control**. Speed still matters — attacker persistence in a vendor tenant frequently migrates into customer tenants via the trust boundary you granted.

## Detection signals

- Direct notification from a vendor ("out of an abundance of caution…") — email, in-app banner, status page, or a coordinated blog post + press release.
- Public disclosure (Bleeping Computer, KrebsOnSecurity, a CVE, an SEC 8-K) about a service you use, before you've been personally notified.
- Alerts from external sources: a customer forwards you a suspicious email that appears to come from your vendor's domain; a researcher DMs you saying they saw your subdomain in a leaked config.
- Anomalies on **your side** of the integration that only make sense if the vendor was compromised: unfamiliar OAuth grants issued by the vendor's service principal, webhook deliveries from unexpected source IPs, `X-Signature` headers using a signing key you didn't rotate but that fails verification, cloud audit entries showing the vendor's role assuming into an account it doesn't normally touch.
- Your SSO IdP (Okta / Entra / Google) shows the vendor's service principal generating tokens for accounts or scopes it has never used before.
- An email from a customer of *yours* saying "your vendor emailed me directly" — a strong tell that the vendor's customer list leaked.

## First 15 minutes

The instinct is to yank every integration. Resist it — for most vendor breaches, ripping the connection blinds you to the ongoing attack and takes a business system down. The First 15 is about **preserving evidence, containing the specific tokens you control, and getting the right people in the room**.

1. **Open the [Incident Record](../templates/incident-record.md).** Note the exact source of the report (vendor email URL, status page snapshot, CVE, news article) and the timestamp *you* received it. The vendor's timeline and yours will differ; both matter for notification obligations.
2. **Snapshot the vendor's public statement** — full-page PDF, not just a link. Vendor statements get edited (and sometimes deleted) in the first 48 hours. Save the pre-edit version.
3. **Do not click any links in the vendor email.** Vendor-breach emails are the #1 phishing pretext of the week following a real breach. Confirm the notification independently: navigate to the vendor's status page or dashboard by typing the URL, not clicking. If it's a phone call, hang up and call back on the number you have on file.
4. **Contain what you control, not what they control:**
   - Rotate **your side** of any shared secret: API keys, webhook signing secrets, service-account passwords, SSH deploy keys, OAuth client secrets you issued to them.
   - **Do not** yet revoke OAuth grants the vendor holds on your systems — leaving them live for a few hours preserves attacker-visible telemetry (see Investigation below). Exception: if the vendor tells you specifically that grants were exfiltrated, revoke immediately.
   - Freeze any auto-deploy pipelines that pull from the vendor (CI/CD tokens, artifact registries, package feeds).
5. **Assemble the room:** IC, an engineer who owns the vendor integration, the Data Protection Officer / privacy lead, and — critically — someone who can read the vendor contract. Vendor MSAs and DPAs govern who's obligated to do what by when; that document is now on the critical path.

## Investigation

You have two parallel investigation tracks: **what did they lose** and **what did we lose because of them**.

### Track 1 — what the vendor lost (your data at their site)

You are dependent on the vendor for answers, and vendors are slow, cautious, and often legally muzzled during the first 72 hours. In parallel with pressing them:

- **Ask a written, specific question set** — not "was our data affected?" but:
  - Which of our records, in which tables/objects, were in the accessed store?
  - Was PII / PHI / cardholder / auth-material included?
  - Did the attacker have read, write, or export capability?
  - Were credentials, tokens, or webhook secrets in the exposed data set?
  - What is the earliest known access time and the earliest *possible* access time (there is usually a gap)?
  - Is there evidence of lateral movement into customer tenants (yours specifically)?
  - Which log types can they share with you (raw, redacted, or none)?
- Timestamp every vendor response. Later, notification-obligation calculations require you to prove *when you knew*.
- If the vendor uses sub-processors (they usually do), ask which sub-processor was the compromise vector. That determines your DPA exposure.

### Track 2 — what you lost because of them (their access into your systems)

For every place the vendor authenticates into your environment:

- **List all OAuth applications, service principals, integration users, webhook receivers, API keys, deploy keys, and network-level allow-listed IPs** the vendor holds against your systems. Most orgs *underestimate* this by 2-3×. Use the vendor console + your IdP + your cloud IAM in combination; each shows a different slice.
- Pull audit logs for every one of those identities for the window from **6 hours before the earliest possible vendor-side access** through now. Preserve to a location the vendor cannot reach — bad-day scenario is the attacker uses the vendor's credentials to delete logs in your tenant.
- Look for scope escalation. A vendor integration that normally reads a single Google Workspace calendar and suddenly enumerates `Directory.Read.All` did not do that on its own.
- Cross-check webhook deliveries against the vendor's documented source-IP ranges. Attackers who exfiltrate webhook-signing secrets replay events from their own infrastructure; the signature validates but the source IP is wrong.
- If the vendor pushes code / artifacts to you (CI, package feed, container registry, model weights) — compute hashes of the last 90 days of pushed artifacts and hold them. If the vendor later publishes IOC hashes, you'll want to grep locally without asking permission.

Reference: [MITRE ATT&CK — Trusted Relationship (T1199)](https://attack.mitre.org/techniques/T1199/) and [Supply Chain Compromise (T1195)](https://attack.mitre.org/techniques/T1195/).

## Eradication

Sequence matters here; get it wrong and you destroy the evidence you needed for Track 2.

1. **After you have logs preserved:** revoke every OAuth grant, service principal, integration user, and webhook signing secret the vendor held against your systems.
2. Rotate every credential that could have been in the exposed data set at the vendor. If a Slack webhook URL was in the vendor's config, rotate the Slack webhook. If a GitHub PAT with `repo` scope was in an environment variable at the vendor, revoke and reissue with the narrowest scope that actually works.
3. **Assume secondary compromise via any secret they may have held** — signing keys, JWT signing secrets, database passwords, cloud IAM keys. If it was in an environment variable at the vendor, it's compromised.
4. Re-establish the integration only after: (a) the vendor confirms remediation, (b) you have reviewed their post-incident write-up, (c) you have narrowed the scope of what the integration can do (least privilege — most integrations are over-scoped by default), and (d) you have an alert on the new integration for scope drift.
5. If the vendor is a package / library / container source: pin to a specific pre-incident version, verify checksums against a copy you had before the breach window, and hold the pin until the vendor publishes a signed clean release.

## Recovery

- Do not restore the integration to its previous permission set. The pre-incident configuration is what got you here. Reduce scope, add scope-drift alerts, add source-IP allow-lists where the vendor supports them.
- Add continuous monitoring on the vendor's status page (RSS + a check that fails loud if the page 404s). Many vendors delete incident write-ups after 90 days.
- Watch for the second wave. Attackers who breach a SaaS provider frequently return 2–8 weeks later using persistence they established during the initial compromise, betting that customer attention has moved on.
- Communicate with your own customers on your timeline, informed by counsel — see Notification & legal.

## Notification & legal

You are almost certainly a **processor** for someone else's data (your customers), and the vendor is a **sub-processor** to you. Downstream notification obligations flow to you even though you weren't the party breached.

| Regime | Trigger | Window | Notes for a downstream victim |
|---|---|---|---|
| GDPR Art. 33 | Personal data breach at any point in the chain | 72h to supervisory authority | Clock starts when *you* become aware — not when the vendor was breached. Document when. |
| GDPR Art. 28 / DPA | Sub-processor breach | Per contract | Your DPA with your customer likely commits you to a shorter window (24-48h). Reread it. |
| HIPAA | PHI exposure via a Business Associate | 60 days | The BA notifies the Covered Entity; the CE notifies individuals. Confirm your role. |
| State breach laws (US) | PII of state residents | Varies | Applies to you as data owner even if the breach was at your processor. |
| PCI DSS | Cardholder data at a service provider | Immediate to acquirer / brand | If Stripe / your processor is involved, their program office runs point but expect an investigation. |
| SEC Reg S-K Item 1.05 | Material cybersecurity incident (public co.) | 4 business days from materiality determination | A vendor breach counts. Determination timing is a legal judgment call. |
| Customer contracts (MSA / DPA) | Per contract | Often 24-72h | Frequently the shortest, and the most often missed. |
| Cyber-insurance | Per policy | Often 24-48h | Late notification is a common denial reason. Notify even if you don't yet think you'll claim. |

**Coordination with the vendor's public statement:** you are not obligated to mirror the vendor's messaging or timeline, and you should not wait for their public statement to notify your own customers if the contract requires you to move sooner. Coordinate with counsel; don't get boxed in by the vendor's PR schedule.

**Do not repeat vendor claims verbatim** in your customer notifications. If the vendor's statement later turns out to be understated (this is common), you own what you told your customers.

## Post-incident

- [Post-incident report](../templates/post-incident-report.md) within 72 hours of incident close; a separate customer-facing version, reviewed by counsel, on the customer-contract timeline.
- Update the vendor risk register: reclassify the vendor's risk tier, revisit the DPA, revisit the SIG-Lite / CAIQ answers they gave you at procurement, and confirm whether the vendor now warrants continuous monitoring rather than annual review.
- Add the vendor's incident to the third-party risk-management questionnaires you send out at renewal — real incidents are stronger vendor signals than certifications.
- Review the integration's blast radius: was the vendor over-scoped? Is there an alternative that could do the same job with less access (e.g., pull-based vs push-based, read-only vs read-write, scoped-per-workspace vs tenant-wide)?
- Pen-test the assumption that your other vendors do better. Pick one vendor at random each quarter and dry-run this runbook against it; you will find gaps in your inventory every time.
- If the vendor's breach involved a source-code or CI/CD platform (GitHub, GitLab, npm, Docker Hub, CircleCI, Codecov, Buildkite): trigger a full secret-rotation across every environment that ever ran a build pulling from them, not just the affected repositories. Historical build logs may have leaked env vars.

## Reporting obligations summary

- Internal: leadership, board-notification triggers per your incident charter.
- Regulators per the table above — your regulators, not the vendor's.
- Affected individuals per applicable laws, on your timeline.
- Law enforcement: FBI IC3 if extortion or wire fraud is involved; do not assume the vendor has reported on your behalf.
- Customers: per contract. Coordinate with legal on wording; err toward specificity about **your** data, not toward relaying the vendor's characterization.
- Cyber-insurance carrier: promptly, and preserve all vendor communications for the claim file.
