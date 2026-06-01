# 05 — Web Application Compromise

> An attacker has achieved code execution, file write, or unauthorized data access **inside your customer-facing application** — typically a Node/Python/Java service in front of a database. This is distinct from a cloud-credential incident: the trust boundary that broke is the app's own logic, not the cloud IAM perimeter. RCE in the app process is usually a stepping stone to data exfiltration and credential theft, so the clock is short.

## Detection signals

- WAF or CDN log spikes against a single endpoint, especially POST endpoints accepting JSON / form data, file uploads, image processors, or `?url=` style fetchers.
- Application logs show unexpected error stacks referencing `child_process`, `subprocess`, `Runtime.exec`, deserialization, template rendering, or SSRF-adjacent code paths.
- A new file appears under `public/`, `uploads/`, `static/`, `tmp/`, or anywhere the app process can write. Webshells in 2026 are still usually a single `.js`, `.php`, or `.jsp` file with `eval` / `Function` / `os.system`.
- Outbound connections from the app host to a destination it has never reached before — Pastebin-like services, Discord/Telegram CDNs, anonymizers, IPs in countries you don't serve.
- A database account used only by the app suddenly issues `SELECT * FROM users` or `pg_dump`-shaped queries against tables the app does not normally export.
- New rows in `users` / `admins` / `api_keys` / `webhook_endpoints` tables that no human created.
- Stripe / Auth0 / Supabase shows API calls from an IP that isn't your app's egress IP.
- Customer reports that a feature is leaking data belonging to other tenants (IDOR / SSRF leakage often surfaces first as a customer complaint).

## First 15 minutes

1. **Drop the affected route or service at the edge — not the host.**
   - Cloudflare / CDN: create a temporary firewall rule blocking the exploited endpoint (`/api/upload`, `/render`, whatever) or rate-limit it to near-zero.
   - Load balancer: take the suspect host(s) out of the target group rather than terminating them. You want them online for memory and process state, but unreachable from the internet.
   - **Do not** stop / terminate / autoscale-replace the container in this window. Modern orchestrators will helpfully rotate the host and destroy your evidence.
2. **Freeze the deploy.** Pause CI/CD on the affected service. The attacker may already be inside the deploy pipeline (stolen CI token, malicious commit) and a "hotfix" deploy could re-introduce them.
3. **Snapshot the running process and disk.**
   - Container: `docker commit <id> evidence-<inc-id>` and push to a forensics registry, or copy `/proc/<pid>/{environ,maps,cwd,exe}`.
   - VM: take an EBS / managed-disk snapshot. Do not reboot.
   - Pull the last 6 hours of application logs to a separate evidence store before any log rotation.
4. **Rotate the app's outbound credentials *in the database / KMS*, not just in the app config**: DB password, Stripe restricted key, Supabase service role, SendGrid key, internal service-to-service JWTs. Rotation in the secret store revokes the old value globally; rotating only in the env-var leaves the leaked secret still valid.
5. **Open a private channel and notify IC + product engineering lead.** Web-app incidents need the engineer who wrote the affected code in the room.

Do **not** in this window: redeploy with a fix, run database migrations, paste suspect payloads into Slack (they may contain XSS or trigger Slack's URL unfurler against attacker infra), or "see if it still works."

## Investigation

- **Initial access vector** — work backward from the earliest hostile request:
  - SQL injection / NoSQL injection (look for `' OR 1=1`, `$ne`, `$where`, or unexpected union/order-by tokens in query logs).
  - Insecure deserialization (Java `readObject`, Python `pickle.loads`, Node `node-serialize`).
  - Server-side template injection (SSTI) — Jinja `{{...}}`, Handlebars, Pug.
  - Server-side request forgery (SSRF) hitting internal metadata endpoints (`169.254.169.254`, `metadata.google.internal`, link-local IPv6).
  - File-upload that bypassed MIME / extension checks (a `.jpg.php`, polyglot images, ImageMagick / GhostScript exploits, image resizers with shell-out).
  - Path traversal / arbitrary file read leaking secrets from `/proc/self/environ`, `.env`, kubeconfig.
  - Known CVE in a framework dependency — check the WAF logs for the specific URI / header pattern from the advisory.
  - Stolen developer credential reaching admin endpoints (overlap with [03-business-email-compromise](./03-business-email-compromise.md) and [01-cloud-credential-compromise](./01-cloud-credential-compromise.md) — run those in parallel if relevant).
- **What the attacker did with the access** — list every action chronologically:
  - Files written / read on the host. Diff against the deploy image's expected file list.
  - DB queries issued by the app's role since the suspected start time. Group by query shape; outliers tell the story.
  - Outbound HTTP from the app to the internet — destinations, byte counts, timing.
  - New rows in privileged tables (`users`, `roles`, `api_keys`, `tenants`, `feature_flags`, `webhooks`).
  - Modifications to existing rows that grant access (email-address changes on admin accounts, password-hash overwrites, role escalations).
  - Tokens minted by the app (session JWTs, password-reset tokens, OAuth codes) since the suspected start time — assume any minted in that window are tainted.
- **Persistence** — what would let the attacker back in even after the obvious vuln is patched?
  - Backdoor user / API key planted in the DB.
  - Webhook URL pointing to an attacker server (so future legitimate events leak data).
  - Modified env-var or feature flag that disables auth / logging.
  - Malicious dependency added to `package.json` / `requirements.txt`.
  - Stolen long-lived refresh tokens or "remember me" cookies.
  - A pull-request opened by a stolen developer credential, in the pipeline pending merge.
- **Blast radius** — what data could the app role read? Treat every record reachable by the app's DB grants as *potentially* accessed unless query logs prove otherwise.

Reference: [MITRE ATT&CK T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/), [OWASP Top 10](https://owasp.org/Top10/).

## Eradication

1. **Patch the vulnerability** — not just block the symptom. A WAF rule is a stopgap, not a fix; the next variant of the payload will route around it.
2. **Rebuild the host(s) from a known-good image.** Do not `git pull && restart`. Container images: rebuild from a commit that predates the compromise; verify by hash. VMs: re-provision from infra-as-code.
3. **Rotate every secret the app process could read.** Application DB credentials, Stripe keys, Supabase service-role, SendGrid, internal JWT signing keys, OAuth client secrets. If unsure whether a secret was reachable from the process — rotate it.
4. **Invalidate every session / token minted in the suspect window.** Force re-login. For JWTs without a server-side store, rotate the signing key so all in-flight tokens fail signature verification.
5. **Remove every persistence artifact found in investigation** — DB rows, dependencies, webhooks, env-vars. Document each one in the incident record with the artifact and the source of truth that should replace it.
6. **Audit the CI/CD pipeline** — review pending PRs, recent merges to `main`, recent secret rotations, recent webhook registrations on the repo. If a developer credential was implicated, treat the pipeline as compromised until proven clean.

## Recovery

- Re-add the host to load balancing only after the rebuild + secret rotation are verifiably complete.
- Keep the WAF rule in place for at least 30 days even after the patch ships — defense in depth, and an early-warning signal if the attacker probes for a regression.
- Heightened monitoring on the affected endpoint: log every request body for 14 days, alert on anomalies.
- Verify customer-facing functionality end-to-end. Eradication often breaks adjacent features that depended on the same secret or the same code path.
- Communicate to the engineering team specifically what the root cause was. The patch is only durable if the team understands the vuln class.

## Post-incident

- [Post-incident report](../templates/post-incident-report.md) within 72 hours.
- Add a regression test that reproduces the original exploit and fails on the pre-fix code. This is non-negotiable for web-app vulns — without a test, the next refactor brings the bug back.
- Threat-model the affected feature properly using the [threat-modeling-framework](https://github.com/batuhan-satilmis/threat-modeling-framework) STRIDE template. The fact that the vuln shipped means the original threat model missed it (or never existed).
- Review the dependency graph. If the root cause was a transitive dependency, enable SCA (Snyk, Dependabot, GitHub Advanced Security) on the repo and gate releases on critical-severity advisories.
- Review egress controls. Most production app hosts have no business reaching arbitrary internet — restrict outbound to an allow-list of legitimate destinations (Stripe, your DB, your queue).
- Review least-privilege on the app's DB role. The blast radius is the role's reachable data; tighten the grants.

## Reporting obligations

- **Customer data accessed**: contractual notification windows (often 24–72h). The same triggers as in [04-data-exfiltration](./04-data-exfiltration.md) apply once data exposure is confirmed.
- **PHI accessed**: HIPAA breach process (within 60 days; immediately if > 500 affected).
- **EU personal data**: GDPR 72-hour notification to the supervisory authority once a breach is established.
- **Public company**: SEC 4-business-day disclosure once materiality is determined.
- **Cyber-insurance**: per policy, typically 24–48 hours from discovery.

If exfiltration is confirmed, **also open [04-data-exfiltration.md](./04-data-exfiltration.md)** — that runbook focuses on the legal-first scoping work that follows.
