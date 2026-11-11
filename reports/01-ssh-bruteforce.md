# Report 01: SSH brute force (simulated)

## What it detects
Repeated failed SSH logins. Built-in Wazuh rules, no custom rule needed:
- **5710** (level 5): login attempt for a non-existent user.
- **5712** (level 10): multiple failed logins in a short time (brute force).

MITRE ATT&CK tags shown by Wazuh: T1110.001 (Password Guessing), T1021.004 (SSH).

## Why it matters
Password guessing against SSH is one of the most common internet-wide attacks.
Repeated failures from one source are the typical first sign.

## Evidence

!(../screenshots/bruteforce_attempts_sc.png)

!(../screenshots/alerts_sc.png)

## What an analyst would do next
1. Find the source IP and check whether it is expected (internal scanner, admin typo) or external.
2. Check whether any attempt succeeded (look for accepted-login events from the same source).
3. If external and persistent: block the source (firewall / fail2ban) and review exposure of port 22.
4. Confirm password authentication is disabled and the targeted usernames do not exist.
