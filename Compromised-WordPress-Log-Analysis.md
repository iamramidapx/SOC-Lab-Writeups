# BTLO Lab: Log Analysis — Compromised WordPress

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

**4. What plugin was actually exploited to gain access?**

**Simple File List.**

Attackers can't achieve RCE with a plain `GET`, so the search focused on
`POST` requests to plugin directories, then filtered to just the attacker's
IP once it was known.

Use: Bash
```
grep "POST" access.log | grep "/wp-content/plugins/"
```
How to verify: Look at the resulting URLs. You will see lines pointing
directly to a specific plugin's folder and processing script (for example,
containing `simple-file-list` or the specific action endpoint of the
vulnerable plugin).

**5. What is the name of the PHP web shell?**

**`fr34k.php`**, sitting in `/wp-content/uploads/simple-file-list/` — found by
grepping for `.php` hits inside the uploads directory.

Use: Find it in the Log Files (access.log)
If you want to search your web server logs to see where and when this
`.php` file was accessed or uploaded:

Bash
```
grep "\.php" access.log | grep "uploads"
```
How to verify: Look at the output lines for any `.php` file located inside
the uploads directory (since standard WordPress uploads shouldn't normally
contain executable PHP files).

**6. What HTTP status code was returned the final time the web shell was accessed?**

**404.**

`grep "fr34k.php" access.log | tail -n 5` shows the most recent hits; the last
line's status code answers this. A `404` here indicates the shell was gone by
that point (removed/cleaned up) rather than still being actively served.

Use: To find the HTTP response code when the web shell (`fr34k.php`) was
accessed for the final time, you can search the log and look at the last
entries using the `tail` command in bash:

Bash
```
grep "fr34k.php" access.log | tail -n 5
```
How to verify: Look at the final line of the output. In standard
Apache/Nginx log formats, the HTTP response code is the 3-digit number
located right after the request path and protocol (usually right before the
byte size).

Example line: `... "GET /wp-content/uploads/fr34k.php HTTP/1.1" 200 4210`

In this example, `200` is the HTTP response code (meaning the file was
successfully accessed and executed). If it shows `404`, it means the file
was not found; if `403`, access was forbidden. Check the very last line of
your command output to get your exact 3-point answer.

