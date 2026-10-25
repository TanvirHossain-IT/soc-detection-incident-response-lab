# Incident 1 — Encoded PowerShell (benign positive)

**Scenario:** An intentionally encoded PowerShell command printed a test marker; no payload download, persistence, or unauthorized action occurred.

- **Source:** Sysmon Event ID 1 ingested into Microsoft Sentinel `Event` table.
- **Detection:** Custom scheduled KQL analytics rule `SOC - Suspicious PowerShell Execution` (Medium severity; MITRE ATT&CK T1059.001).
- **Host:** Windows 11 lab VM (hostname omitted from the public report).
- **Activity:** `powershell.exe -NoProfile -EncodedCommand <base64>`; decoded command printed `SOC-LAB-ANALYTICS-RULE-TEST`.
- **Evidence:** Test process event at **2026-10-25 02:13:37 UTC**, automatically generated security alert at **02:23:02 UTC**; one matching event in alert; host entity was mapped.
- **Triage:** Validated process image, full command line, parent context, and lab authorization.
- **Disposition:** Incident #1 closed **Benign Positive — Suspicious but expected**, with closure notes.
- **Lesson:** Alert creation and host mapping were verified end to end. Encoded execution alone is not proof of malicious intent.
