# Lab Journal

Record each hands-on session with the actual date, tools, actions, evidence, results, errors, and next step. Do not claim an outcome until it is verified.

## Session 01 — Windows Event Viewer
**Date:** October 22, 2026 (local)

**Status:** Foundational Windows log review complete; phase transitioned to dedicated test VM.

**Objective:** Locate the Windows Security log, interpret basic event records, and test a failed-logon event filter.

**Completed procedure:**
1. Opened Windows Event Viewer.
2. Expanded `Windows Logs` and selected `Security`.
3. Inspected Event ID 5379 and captured a screenshot.
4. Applied the `Filter Current Log` filter for Event ID `4625` (failed logon) and captured a screenshot of the results.
5. Ran `auditpol /get /subcategory:"Logon"` in a standard Command Prompt; Windows returned `0x00000522` (required privilege not held).
6. Re-ran the command using an elevated Administrator Command Prompt, verifying `Logon  Success and Failure`.
7. Filtered the Security log for Event ID `4624` and inspected one successful logon with Logon Type `5`.

**Evidence:**
- `screenshots/01-windows-event-logs/01-security-event-5379-redacted.png` (personal account and host information redacted)
- `screenshots/01-windows-event-logs/02-security-filter-4625-zero-events.png` (filtered result, no personal account detail shown)
- `screenshots/01-windows-event-logs/03-audit-logon-success-failure.png` (elevated audit policy output)
- `screenshots/01-windows-event-logs/04-security-filter-4624-successful-logons.png` (1,673 matching 4624 records)
- `screenshots/01-windows-event-logs/05-service-logon-4624-properties-redacted.png` (redacted 4624 record header, full fields provided in chat)

**Verified observations:**
- The Security log displayed **32,707 events** in the first capture.
- The selected entry had **Event ID 5379**, described as **Credential Manager credentials were read**, with **Audit Success**, timestamped **10/22/2026 11:09:39 PM**.
- When the failed-logon filter `4625` was applied, Event Viewer displayed **32,725 total events** and **0 matching events**.
- These screenshots do **not** establish that no failed logons ever occurred. Auditing settings, log retention, and log scope limit the conclusion.
- Audit policy output verifies Windows has both successful and failed **Logon** auditing enabled at the time of the check. This does not establish that the event history includes failed logons.
- The Security log showed **1,673 Event 4624** (successful logon) records out of **32,716** total events when filtered on October 22, 2026.
- An inspected event at **11:17:50 PM** had **Logon Type 5** (service logon), **New Logon account `NT AUTHORITY\SYSTEM`**, **Subject `SYSTEM`**, **process `C:\Windows\System32\services.exe`**, **Elevated Token: Yes**, and **no network source recorded**. This is consistent with an ordinary local Windows service logon; there is no indication of malicious activity from this record alone. Elevated token and Logon Type 5 are expected in this context.
- No malicious activity has been established by the available observations.

**Troubleshooting:** The initial `auditpol` command returned `Error 0x00000522: A required privilege is not held by the client` when run without elevation. Reopening Command Prompt as administrator resolved the issue and returned the audit policy.

**Additional follow-up:** Filtered 4624 records by interactive Logon Type 2 (16 events), examined normal `DWM-1` and `DWM-5` virtual accounts, and found zero exact matches using the queried `TargetUserName` value. The lack of matches does not prove a lack of legitimate user logons.

**Next step:** Continue with the dedicated VM; keep host and account details out of public evidence. Avoid repeated password attempts against personal or work accounts.

## Session 02 — Windows VM & Sysmon
**Date:** October 23, 2026 (local)

**Status:** Phase 2 complete: VM running, both snapshots created, Sysmon configured, process/DNS/network telemetry verified.

**Detailed notes:** [`02-WINDOWS-VM-SYSMON.md`](02-WINDOWS-VM-SYSMON.md)

**Additional completed work:** Inspected Sysmon's initial settings; added an XML event configuration and validated its successful application. Verified `Resolve-DnsName example.com` logged Sysmon Event 22 and `Test-NetConnection -ComputerName example.com -Port 443` logged Sysmon Event 3 associated with PowerShell. Created a second VirtualBox snapshot named `SOC-WIN11-Sysmon-Configured`. Evidence: `screenshots/02-sysmon/10-` through `15-`, plus `config/sysmon-config.xml`.

**Next:** Phase 3 — check Azure subscription availability and cost controls, then plan Sentinel ingestion. No paid Azure resources have been created in this project.


## 2026-10-24–25 — Phase 3 Sentinel integration completed
- Confirmed Azure for Students $100 student credit offer at setup.
- Created `SOC-Lab-Budget` monthly $5 with alerts at 50%, 80%, 100%; configured Log Analytics daily cap 0.05 GB/day.
- Resolved `RequestDisallowedByAzure` region policy error by deploying in approved **North Central US**.
- Deployed `rg-soc-lab`, `law-soc-lab-01`, and Sentinel (31-day trial confirmed in UI).
- Connected VirtualBox Windows endpoint to Azure Arc (Connected), deployed AMA (Succeeded), and associated DCR `dcr-soc-sysmon` using Sysmon XPath filter for IDs 1, 3, 22.
- Verified Microsoft Sentinel Logs `Event` table results: **171 Sysmon events** (13 / 142 / 16). Phase 3 complete.
- Azure resources survive shutting down the VM, but agent telemetry stops until the VM boots and reconnects. No third VM snapshot was taken.
- Next: Phase 4, KQL detection engineering and benign alert tests.


## 2026-10-25 — Phase 4 KQL and first automated incident
- KQL baseline: 728 Sysmon Event records (80 Event 1, 611 Event 3, 37 Event 22).
- Saved queries for PowerShell patterns, DNS-process correlation and outbound connections.
- Created the first scheduled PowerShell analytics rule; test event matched, alert/Incident #1 generated, host entity mapped, closed as benign positive.
- Details: `04-KQL-DETECTION-ENGINEERING.md`; case study `../reports/01-encoded-powershell.md`.

## 2026-10-25 — Phase 5 discovery, DNS and Rundll32
- Simulated safe `whoami` and `ipconfig` discovery; Sysmon showed process tree and automated Incident #2; reviewed and closed benign positive with comment.
- Simulated three unsuccessful `soc-test-*` DNS queries using `Resolve-DnsName`; logged all three, grouped into one KQL match and Incident #3; reviewed and closed benign positive. Rule host entity mapping subsequently corrected; requires fresh-alert validation before claiming fix is proven.
- Ran two benign `rundll32.exe` commands to launch Control Panel and Programs and Features, compared baseline and tuned hunts; no new automated incident from these tests.
- Completed draft report with individual case studies in `reports/`.
- Next: Phase 6 publication prep and verify cost/resource cleanup when lab work ends.
