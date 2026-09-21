# BTLO Lab: Log Analysis — Compromised WordPress

**Platform:** Blue Team Labs Online (BTLO)

**Skill area:** Log analysis, Incident Response (IR), Linux command-line investigation

## Scenario

A WordPress site was compromised. The working hypothesis: an installed plugin
had a remote code execution (RCE) vulnerability that let an attacker reach the
underlying OS. The only evidence available is the web server's `access.log`
(Apache/Nginx combined log format). The goal is to reconstruct the attack
using nothing but `grep`, `awk`, and `find`.


## Command cheat sheet

**Core idea:** log analysis is about picking the right *keyword* — ask "what
would this specific behavior leave behind in the log?" — then feeding that
word to `grep`. Getting the concept right matters far more than memorizing
exact syntax.

## Investigation walkthrough

**1. What URI did the attacker use to reach the admin login (including the token)?**

`/wp-login.php?itsec-hb-token=adminlogin`

This site hid its login behind a security-plugin token instead of the default
`/wp-login.php`. Found by grepping for `token` / `wp-login`, then confirming
success by looking for a `POST` to that URI followed by an HTTP `302`
(redirect into the dashboard).

Use: Bash
```
grep "token" access.log
```
e.g. `POST /wp-login.php?token=xxxxxxxxxxxx HTTP/1.1" 302 ...`

**2. What two tools did the attacker use?**

**WPScan** and **sqlmap**.

Attacker tooling usually leaves a fingerprint in the `User-Agent` field.
Ranking all User-Agents with the `awk | sort | uniq -c | sort -nr` combo
surfaces anything that isn't a normal browser string — automated scanners
show up immediately (e.g. `WPScan v3.8.22`, `sqlmap/1.6.8#dev`,
`python-requests/...`).

Use:
```
grep -iE "wpscan|sqlmap|nmap|nikto|python|ruby" access.log
```

**3. What CVE was the exploited "Contact Form 7" component vulnerable to?**

**CVE-2020-35489** — an arbitrary file upload vulnerability in the "Drag and
Drop Multiple File Upload" add-on for Contact Form 7, which allowed
uploading `.phar` or `.svg` files past the filter (handled server-side by
`dnd-upload-cf7.php`). Knowing the exact bypass (which extensions) gave a
concrete keyword to grep for in the log.
