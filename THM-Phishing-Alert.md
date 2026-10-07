# Analyst Simulation – Case Summary

Write-up of a SOC analyst training simulation (fictional company **The Try Daily**, domain `thetrydaily.thm`). All hosts, users and domains are simulated lab data.

## Scenario

Play a SOC analyst: take alerts from the queue, investigate in the SIEM (webserver/ModSecurity, email and firewall logs), check indicators in a threat-intel tool, classify each alert, and write a case report.

**Triage workflow**
1. Review the alert queue and take the earliest alert.
2. Read the alert logic and IOCs (IPs, domains, URLs).
3. Query the SIEM for related logs and build a timeline.
4. Check indicators in the threat-intel app.
5. Classify, write the report, and decide on escalation.

**Classification**
- **True Positive (TP):** real malicious activity (even if blocked or unsuccessful).
- **False Positive (FP):** harmless activity that triggered the rule.
- **Benign Positive:** real but authorised activity (e.g. admin vulnerability scan).

**Escalate** a TP when remediation is needed or it is part of a larger attack chain. No escalation if it was contained before any impact (e.g. quarantined email, installer removed before execution).

## Case Report Template

| Field | Content |
|---|---|
| Time of activity | Exact timestamp from the alert |
| Affected entities | Who/what/where (user, host, IP) |
| Reason for classification | Evidence-based TP/FP rationale |
| Reason for escalation | Why remediation or follow-up is needed (or N/A) |
| Remediation actions | Concrete next steps |
| Indicators (IOCs) | URLs, domains, IPs, ports, sender addresses |

Tip: use the **W5** framework: Who, What, Where, When, Why.

## Environment

- Office network: `10.20.2.0/24`, 13 employees on hosts `win-3451` to `win-3463`
- Log sources: webserver (ModSecurity), email, firewall
- Relevant mapping: `10.20.2.17` = Hannah Harris (HR), `win-3457`

## Alerts Triaged

| # | Alert | Time (08/24/2026) | Target | Verdict | Escalate |
|---|---|---|---|---|---|
| 1 | Inbound email with suspicious link: "Finalize Your Onboarding Profile" from `onboarding@hrconnex.thm` | 09:39:04 | j.garcia | See note below | Pending firewall check |
| 2 | Same onboarding email (re-send) | 09:45:03 | j.garcia | See note below | Pending firewall check |
| 3 | Firewall: access to blacklisted URL blocked, `http://bit.ly/3sHkX3da12340` | 09:43:31 | 10.20.2.17 → 67.199.248.11:80 | **True Positive** | Yes |
| 4 | Email: fake Amazon "package couldn't be delivered" from `urgents@amazon.biz` | 09:42:17 | h.harris | **True Positive** | Yes |
| 5 | Email: fake Microsoft "unusual sign-in" from `no-reply@m1crosoftsupport.co` | 09:44:35 | c.allen | **True Positive** | Yes |

### Key findings

**Amazon phishing chain (alerts 3 + 4)**
The bit.ly link in the phishing email sent to `h.harris` is identical to the URL the firewall blocked from `10.20.2.17`, which is Hannah Harris's host. Together they show the user clicked the link and the firewall stopped it. Both alerts belong to a single incident, so both are escalated.
- Remediation: purge the email, block `amazon.biz` and the bit.ly URL, scan `win-3457`, check whether other hosts contacted `67.199.248.11`, and give the user phishing-awareness training.

**Microsoft typosquat phishing (alert 5)**
The sender domain `m1crosoftsupport.co` uses a digit "1" in place of "i". The email invents a sign-in from Lagos, Nigeria to create urgency and points to a fake login page (credential harvesting).
- Remediation: purge the email, block the domain at the mail gateway and proxy, and check firewall logs and the user's account for credential submission.

**HR onboarding emails (alerts 1 and 2)**
The notes classify these inconsistently (TP for the first, FP for the second). The sender domain `hrconnex.thm` is unverified, and no firewall evidence of a click is recorded. The deciding evidence is whether the domain is on the approved vendor list or part of a known awareness test, and whether any host contacted the URL. Treat the two as the same case and give them one verdict.

## IOCs

| Type | Value |
|---|---|
| URL | `http://bit.ly/3sHkX3da12340` |
| IP | `67.199.248.11` (port 80) |
| Sender | `urgents@amazon.biz` |
| Sender | `no-reply@m1crosoftsupport.co` |
| Domain | `m1crosoftsupport.co` |
| URL | `https://m1crosoftsupport.co/login` |
| Sender / URL | `onboarding@hrconnex.thm`, `https://hrconnex.thm/onboarding/15400654060/j.garcia` |

## MITRE ATT&CK Mapping

- T1566.002 – Phishing: Spearphishing Link
- T1204.001 – User Execution: Malicious Link
- T1598 – Phishing for Information (credential harvesting)
- T1036 – Masquerading (brand impersonation)

## Lessons Learned

- Correlate across data sources: an email alert plus a firewall alert on the same URL links two alerts into one incident.
- Map internal IPs to users with the asset list before judging impact.
- A **blocked** connection still warrants escalation when it shows a user acted on a phishing email.
- Look for typosquatting (`m1crosoft`), look-alike TLDs (`amazon.biz`) and URL shorteners.
- Be consistent: identical alerts should get the same classification and rationale.
- "Escalation reason" should explain the risk, not just list the checks performed.

