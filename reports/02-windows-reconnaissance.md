# Incident 2 — Windows reconnaissance (benign positive)

**Scenario:** Authorized execution of Windows user and network discovery commands inside the lab VM.

- **Commands:** `cmd.exe /c whoami` and `cmd.exe /c ipconfig`.
- **Source:** Sysmon Event ID 1; four process-creation records in the manual hunt: two `cmd.exe` instances and child processes `whoami.exe`, `ipconfig.exe`.
- **Parent-child process evidence:** `powershell.exe` → `cmd.exe` → `whoami.exe` and `ipconfig.exe`.
- **Detection:** Scheduled analytics rule `SOC - Windows Reconnaissance Detection`, Low severity, targeting child executable names; correlated two child-process matches into one alert.
- **MITRE ATT&CK context:** T1033 (System Owner/User Discovery), T1016 (System Network Configuration Discovery). These are mappings to observed *techniques*, not proof of hostile intent.
- **Evidence:** Incident #2 showed one alert and two contributing event records. Host entity mapping identified the Windows lab VM.
- **Triage:** Confirmed the commands were intentionally executed in a controlled environment.
- **Disposition:** Incident #2 closed **Benign Positive — Suspicious but expected**; investigation comment posted.
- **Lesson:** Parent-child process context helps explain discovery commands and reduce false conclusions.
