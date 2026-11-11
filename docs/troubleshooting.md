# Troubleshooting

Problems I met while building this lab, in the ORDERR they happened.

## 1. Index errors, Indexer stopped writing
- **Symptom:** errors about indexes in the dashboard.
- **Cause:** the VM disk was full; the Indexer switched indices to write-protect.
- **Fix:** I expanded the VM disk from 25 GB to 60 GB.

## 2. "Dashboard server is not ready"
- **Cause:** the Indexer was still starting.
- **Fix:** I waited until the Indexer finished loading.

## 3. API Offline after reboot
- **Cause:** the manager failed to start after the reboot (systemd timeout). Every
  later start failed because of a leftover lock directory `start-script-lock`.
- **Fix:** removed the lock directory and set `TimeoutStartSec=600` for the service.

#### Lesson: never ever press Ctrl+C while the manager is starting

## 4. Events "missing" from the dashboard
- **Cause:** the dashboard shows local time (UTC+5) while files store UTC.
  Also a fixed end time in the time picker hid newer events.
- **Fix:** use relative ranges ("Last 1 hour") and remember the UTC offset.

## 5. Custom rule 100100 did not fire
- **Cause:** FIM scanned `/etc` only every 12 hours. Files created before the
  restart went into the baseline silently, so no "file added" event was produced.
- **Fix:** `realtime="yes"` for `/etc` in the agent's `ossec.conf`
  ([snippet](../agent-config/ossec-syscheck-snippet.xml)), then restarted the agent.

## 6. Custom rule 100101 did not fire
- **Symptom:** changes to `sshd_config` produced only the generic rule 550 (level 7).
- **Cause:** the rule was **not in `local_rules.xml`** (the edit had not been saved).
  I first suspected the `^`/`$` anchors in the path; that was wrong.
- **Fix:** appended the block again and verified it with `grep -c 100101 local_rules.xml`
  before restarting the manager.
  Lesson: after editing a rules file, confirm the content, then `wazuh-analysisd -t`, then restart.

