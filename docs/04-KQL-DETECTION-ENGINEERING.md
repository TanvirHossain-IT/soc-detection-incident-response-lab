# Phase 4 — KQL Hunting and Automated Detection

**Completed and verified:** 2026-10-25 UTC.

- Confirmed 728 recent Sysmon events (ID 1: **80**, ID 3: **611**, ID 22: **37**) at a point-in-time check.
- Investigated benign Windows Update processes, Azure Arc PowerShell maintenance, Windows DNS queries, and common outbound connections.
- Saved query collection: PowerShell execution, DNS process investigation, and network connection investigation.
- Generated a harmless Base64-encoded PowerShell command, used KQL to detect it, and created the **Medium** scheduled rule `SOC - Suspicious PowerShell Execution` mapped to **T1059.001**.
- Verified Sentinel Incident **#1**, its host entity, one matching alert, supporting event, and closed it **Benign Positive — Suspicious but expected**.
- Distinguish detection of an execution *pattern* from proof of an actual security compromise.

**Reference queries:** `queries/phase4-*.kql`. **Evidence:** `screenshots/04-detections/`.
