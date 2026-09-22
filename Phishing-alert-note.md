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

--------

# BTLO — Phishing Analysis (Short Summary)

[Challenge Link](https://blueteamlabs.online/home/challenge/phishing-analysis-f92ef500ce)

## Answer 

| # | Question | Answer |
|---|---|---|
| - | Primary recipient | `kinnar1975@yahoo.co.uk` |
| - | Subject | Undeliverable: Website contact form submission |
| - | Date/Time sent | 18 March 2021 04:14 |
| 1 | Originating IP | `103.9.171.10` |
| 2 | Reverse DNS host | `c5s2-1e-syd.hosting-services.net.au` |
| 3 | Attached file name | `Website contact form submission.eml` |
| 4 | URL inside attachment | `https://35000usdperwwekpodf.blogspot.sg?p=9swghttps://35000usdperwwekpodf.blogspot.co.il?o=0hnd` |
| 5 | Webpage hosting service | Blogspot |
| 6 | Heading text (URL2PNG) | `Blog has been removed.` |

## Key Findings from the Actual .eml File

- **Originating IP** was found in the first `Received:` header inside the embedded attachment (message/rfc822):
  ```
  Received: from c5s2-1e-syd.hosting-services.net.au ([103.9.171.10])
  ```
- The original submission came from a contact form on `www.markgardner.com.au`, submitted from IP `91.90.123.43`.
- The `X-AntiAbuse` headers reveal sender domain spoofing (`Original Domain: yahoo.co.uk`, `Sender Address Domain: hotmail.co.uk`).
