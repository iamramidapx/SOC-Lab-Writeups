# BTLO Lab Summary: Log Analysis – Compromised WordPress

## Scenario
A WordPress site was compromised and the cause was unknown. **Hypothesis:** a vulnerable plugin allowed **Remote Code Execution (RCE)**, giving the attacker access to the server's operating system.
**Evidence available:** the web server `access.log` (Apache/Nginx). It is a plain text file, so Linux command-line tools can search it.

## Core idea (the method)
Log analysis is detective work. Ask: **"What would this attacker action leave behind in the log?"** Then `grep` for that trace, and filter out normal visitor traffic.

1. What am I looking for? (a keyword, IP, file name or status code)
2. What can I throw away? (normal browsers and normal requests)
3. How should the output look? (a count, a column, the last lines)

This maps to the **Incident Response (IR)** lifecycle: Detection & Analysis → Containment → Eradication → Recovery & Lessons Learned. The lab exercises the first phase, finding the root cause.

---

## Why these tools?

| Tool | What it does | Why it is used here |
|---|---|---|
| `grep` | Searches for text inside a file (like Ctrl+F, but for GB-sized files) | The log is a text file. Attacker actions leave keywords such as `POST`, `wp-login`, `.php`, `wpscan` and an IP. |
| `\|` (pipe) | Feeds one command's output into the next | Filters in layers, so results get narrower and more precise. |
| `grep -iE "a\|b\|c"` | `-i` ignores case, `-E` allows "or" patterns | Searches for several suspicious words in one pass. |
| `awk` | Extracts a specific column or field | `$1` is the IP address, and `-F\"` with `$6` is the User-Agent. |
| `sort` + `uniq -c` + `sort -nr` | Groups duplicates, counts them, ranks by frequency | Rare or odd User-Agents and IPs stand out against normal traffic. |
| `tail -n 5` | Shows the last 5 lines | The final access to something is the newest log line. |
| `less` | Scrolls long output page by page | Keeps tens of thousands of lines readable. |
| `find` | Finds files by name, time (`-mtime`, `-mmin`), permission (`-perm 0777`) | Used on the server's file system, not the log, to find dropped web shells such as `.php` files in `uploads/`. |

**Rule of thumb:** text inside a file → `grep`; the file itself → `find`; a particular column → `awk`.

---

## Questions and answers, with reasoning

### Q1. Admin login URI the attacker used (including token)
**Answer:** `/wp-login.php?itsec-hb-token=adminlogin`

- **Tool and why:** `grep "wp-login" access.log` and `grep "token" access.log`. The question names a login page and a token, so those are the keywords the request must contain.
- **Why this is the answer:** a successful login shows as a **POST** to the login URI with status **302**. A 302 is the redirect to the dashboard. The URI that the 302 POST hit, including the `itsec-hb-token` parameter (an iThemes Security "hide backend" token), is the answer.

### Q2. Two tools the attacker used
**Answer:** **WPScan** and **sqlmap**

- **Tool and why:** `awk -F\" '{print $6}' access.log | sort | uniq -c | sort -nr | less`. Attackers do not announce "I'm using a tool", but automated tools usually leave their own name in the **User-Agent**. This command lists every User-Agent with a count.
- **Why this is the answer:** normal visitors show `Mozilla/5.0 … Chrome/Safari`. Entries such as `WPScan v3.x` (a WordPress scanner) and `sqlmap/1.x` (a database-injection tool) do not look like browsers. Those are the 🚩 entries. A direct search also works: `grep -iE "wpscan|sqlmap|nmap|nikto|python|ruby" access.log`.

### Q3. CVE for the Contact Form 7 vulnerability
**Answer:** **CVE-2020-35489**

- **Method:** OSINT. The log shows the plugin name, and the CVE comes from external research on that plugin's vulnerability. The log alone cannot give a CVE number.
- **Why this is the answer:** the CVE description reveals the exploit's **behavior**. It is an unrestricted file upload (`.phar`/`.svg`, handler `dnd-upload-cf7.php`, upload path `wpcf7_drag_n_drop_uploads`). That behavior gives new `grep` keywords to take back to the log: `grep -iE "\.phar|\.svg|wpcf7|dnd-upload" access.log`.
- **Workflow:** get a clue from the log → research the CVE → learn how the exploit works → search the log again with those keywords.

### Q4. Plugin exploited to gain access
**Answer:** **Simple File List**

- **Tools and why:** `grep "POST" access.log | grep "/wp-content/plugins/"`, then narrow to the attacker's IP with `grep "103.69.55.212" access.log | grep "POST"`. Uploading a file or triggering RCE needs a **POST**, because a GET only views a page.
- **Why this is the answer:** the folder name right after `/wp-content/plugins/` (or `/uploads/`) in the attacker's POST requests is `simple-file-list`.
- **Note:** Contact Form 7 was probed (Q3), but the path of the working shell is under `/uploads/simple-file-list/`. This shows attackers can probe several plugins, and the one that actually delivered the shell is the answer.

### Q5. Name of the PHP web shell
**Answer:** `fr34k.php`

- **Tools and why:** `grep "\.php" access.log | grep "uploads"` on the log, and `find /var/www/html/wp-content/uploads/ -name "*.php"` on the server. The `uploads/` folder should hold only images and documents, so a `.php` file there is almost certainly a **web shell/backdoor**.
- **Why this is the answer:** the file `fr34k.php` was requested from `/wp-content/uploads/simple-file-list/` with status 200, meaning it was uploaded and executed. `grep "fr34k.php" access.log` shows the earlier POST (upload) followed by GET requests (execution).

### Q6. HTTP response code on the web shell's final access
**Answer:** **404**

- **Tool and why:** `grep "fr34k.php" access.log | tail -n 5`. The final access is the last matching line, and `tail` shows only the end of the file.
- **Why this is the answer:** the status code is the 3-digit number after the request line. The last line for `fr34k.php` shows `404` (Not Found). The shell was no longer available, which suggests it had been removed or the attack had ended.

---

## Attack story reconstructed
1. **Recon:** the attacker (IP `103.69.55.212`) scanned with **WPScan** and probed with **sqlmap**.
2. **Login:** the attacker reached the admin login via `/wp-login.php?itsec-hb-token=adminlogin` (POST → 302).
3. **Initial access:** the attacker abused the **Simple File List** plugin upload. Contact Form 7 (CVE-2020-35489) was also targeted.
4. **Persistence/RCE:** the attacker uploaded the web shell **`fr34k.php`** and executed it (GET → 200).
5. **End:** the final request to the shell returned **404**.

## Useful command cheat sheet
```bash
grep "/wp-content/plugins/" access.log                          # plugin activity
grep "POST" access.log | grep "/wp-content/plugins/"            # uploads / payload delivery
grep -iE "whoami|wget|curl|bash|cmd|base64" access.log          # RCE signatures
grep -iE "wpscan|sqlmap|nmap|nikto|python|ruby" access.log      # scanner User-Agents
awk -F\" '{print $6}' access.log | sort | uniq -c | sort -nr    # User-Agent ranking
grep "103.69.55.212" access.log                                 # everything by one IP
grep "fr34k.php" access.log | tail -n 5                         # last accesses of shell
grep -i "wpscan" access.log | awk '{print $1}' | sort | uniq -c | sort -nr   # IPs using a tool
find /var/www/html -type f -mtime -3                            # recently changed files
find /var/www/html/wp-content/uploads/ -type f -name "*.php"    # PHP in uploads
find /var/www/html -type f -perm 0777                           # dangerous permissions
```
**Safety:** never open a suspected web shell in a browser. Read it with `cat` or `less`.

## Key takeaways
- Think in **keywords**: what would this action write into the log?
- **POST** means upload or delivery. **GET** means viewing or executing. **302** after a login POST means success. **200** means success. **404** means not found.
- Normal-looking traffic is the majority, so **count and rank** to expose the odd entries.
- Combine **log evidence** (grep) with **external research** (CVE/OSINT), then go back to the log with new keywords.
- You do not need to memorize the commands. Understand the concept, keep a cheat sheet, and build the command from it.
