# Case Study 4 — Rundll32 execution and detection tuning

**Scenario:** Two safe commands launched legitimate Windows Control Panel interfaces:

```powershell
rundll32.exe shell32.dll,Control_RunDLL
rundll32.exe shell32.dll,Control_RunDLL appwiz.cpl
```

- **Telemetry:** Sysmon Event ID 1 reported `rundll32.exe` as the process and `powershell.exe` as the parent.
- **Baseline hunt:** Identified the first execution and then the `appwiz.cpl` variant in Sentinel Logs, inspecting full paths and command lines.
- **Initial tuning:** Added an exclusion for the exact first invocation from PowerShell; observed one match for the second variation.
- **Final tuning:** Excluded both specific command lines when launched by PowerShell; query returned **0 matches** for the captured lab samples.
- **MITRE ATT&CK context:** T1218.011 (Rundll32) is relevant to abuse of this Windows utility. **Our tests do not demonstrate actual defense evasion**; they show benign usage of an abuse-prone binary and a tuning exercise.
- **Disposition:** Benign observation; **no automated Rundll32 incident was created or closed** in this exercise.
- **Limitation:** Exact command-line exclusions are narrowly scoped to the tests and can hide misuse of those same invocations; they are **not production-ready allowlists**. Rules need broader validation and additional context.
