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
