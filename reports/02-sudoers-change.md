# Report 02: Sudoers drop-in change (rule 100100)

## What it detects
A file added or modified in `/etc/sudoers.d/`. The rule builds on built-in
FIM rules 550 (checksum changed) and 554 (file added) and raises the level
to 12 (High) when the path matches.

```xml
<rule id="100100" level="12">
  <if_sid>550, 554</if_sid>
  <field name="file">/etc/sudoers.d/</field>
  <description>Sudoers drop-in file added or modified</description>
  <mitre><id>T1548.003</id></mitre>
</rule>
```

## Why it matters
Files in `/etc/sudoers.d/` define who may run commands as root. An attacker who
can write there can grant themselves administrator rights and keep them
(privilege escalation, persistence). MITRE: T1548.003 (Sudo and Sudo Caching).
Without the custom rule the same event is only a generic level 7 alert.

## How I tested
On the monitored server I created a harmless test file. Sudo ignores files whose
name contains a dot, so it cannot change privileges:

```
sudo touch /etc/sudoers.d/test4.bak
```

Alert 100100 (level 12) appeared within seconds. Path in the alert: `/etc/sudoers.d/test4.bak`,
event `added`, mode `realtime`. The test file was removed afterwards.

**Lesson:** my first attempts (create file, restart agent) produced no alert.
FIM scans `/etc` only every 12 hours; a restart just rebuilds the baseline.
Enabling `realtime="yes"` for `/etc` fixed it (see [troubleshooting](../docs/troubleshooting.md)).

## Evidence
[custom_rules](../rules/local_rules.xml)

## What an analyst would do next
1. Who made the change? Check sudo log events (rule 5402) around the same time.
2. Read the file content: does it grant `NOPASSWD` or ALL rights to an unexpected user?
3. CHeck if there was a change ticket? If not, treat as suspicious: remove the file and look for other persistence.
4. Just review the logins then
