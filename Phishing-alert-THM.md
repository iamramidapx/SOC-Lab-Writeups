# Alert Case Note

## 1. Alert Summary

This alert was triggered by an inbound email containing one or more external links with potentially suspicious characteristics. As part of the investigation, check firewall or proxy logs to determine whether any endpoints attempted to access the URLs in the email, and whether those connections were allowed or blocked.

- **Category:** Phishing
- **Severity:** Medium
- **Alert Time:** Aug 24th 2026 at 15:41

## Alert Details

| Field | Value |
|---|---|
| Data Source | email |
| Timestamp | 08/24/2026 09:39:04.574 |
| Subject | Action Required: Finalize Your Onboarding Profile |
| Sender | onboarding@hrconnex.thm |
| Recipient | j.garcia@thetrydaily.thm |
| Attachment | None |
| Direction | Inbound |

**Content:**
> Hi Ms. Garcia,
>
> Welcome to TheTryDaily!
>
> As part of your onboarding, please complete your final profile setup so we can configure your access.
>
> Kindly please click the link below:
>
> [Set Up My Profile](https://hrconnex.thm/onboarding/15400654060/j.garcia)
>
> If you have questions, please reach out to the HR Onboarding Team.

---

## Investigation Summary — True Positive

**Time of Activity:** 08/24/2026 09:39:04.574

**Affected Entities:**
- j.garcia@thetrydaily.thm

**Reason for Classifying as True Positive:**
The inbound email contains a suspicious external link (`https://hrconnex.thm/onboarding/15400654060/j.garcia`) disguised as an HR onboarding profile setup, matching common credential harvesting and spear-phishing patterns.

**Reason for Escalating the Alert:**
Verified proxy and firewall logs to check if the endpoint (j.garcia) attempted to access the malicious URL, and confirmed whether the connection was allowed or blocked.

**Recommended Remediation Actions:**
- Block the sender domain (`hrconnex.thm`) and the target URL at the email gateway and web proxy levels
- Purge the malicious email from the user's inbox

**Attack Indicators (IOCs):**
- Sender Address: `onboarding@hrconnex.thm`
- Malicious URL: `https://hrconnex.thm/onboarding/15400654060/j.garcia`

