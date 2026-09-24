# BTLO — Phishing Analysis (Summary)

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
