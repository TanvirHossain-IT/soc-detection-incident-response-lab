# Lab 02 — Windows VM & Sysmon Process Monitoring

**Date:** October 23, 2026 (local; Oct 24 UTC timestamps may appear in Sysmon)

**Status:** Phase 2 completed: VM, Sysmon configuration, process/DNS/network event tests, and second snapshot verified. Microsoft Sentinel ingestion is not yet configured.

## Objective
Create a dedicated Windows test endpoint, establish a recovery snapshot, install Sysmon, and verify that process-creation telemetry captures a deliberately launched child process and its ancestry.

## Environment built
- **Hypervisor:** Oracle VirtualBox.
- **VM identifier:** `SOC-WIN11-01`.
- **OS:** Windows 11 Enterprise Evaluation, 64-bit.
- **Resources:** 8,192 MB RAM, four virtual CPUs, dynamically allocated 80-GB VDI.
- **Boot:** UEFI/EFI, Secure Boot, TPM 2.0 enabled.
- **Network:** NAT used for initial setup and updates. NAT is *not complete isolation*. Before more sensitive test simulations, review the network design and use an isolated lab network as appropriate.
- **Host integration:** VirtualBox Guest Additions 7.2.16 installed; user confirmed shared clipboard and drag-and-drop disabled; no shared folders were listed.
- **Recovery:** `SOC-WIN11-Clean-Install` snapshot created after guest OS setup and Guest Additions, before Sysmon.

## Procedure and results
1. Installed Windows 11 Enterprise Evaluation from an official ISO after resolving an initial unsuccessful boot by mounting the ISO.
2. Finished Windows initial setup and updates. Confirmed Windows desktop reachable.
3. Installed Guest Additions and rebooted.
4. Reviewed the VM network mode (NAT), sharing options, and created a baseline snapshot.
5. Downloaded and extracted Sysmon from Microsoft Sysinternals.
6. Opened **PowerShell as Administrator**, navigated to the extracted folder, and ran:

   ```powershell
   .\Sysmon64.exe -accepteula -i
   ```

7. Installation reported the Sysmon service and driver installed and started successfully. Sysmon identified itself as **v15.22** at installation.
8. Confirmed `Applications and Services Logs > Microsoft > Windows > Sysmon > Operational` was recording events. Observed event IDs **1** (process created) and **5** (process terminated).
9. Launched Notepad as a controlled benign test with:

   ```powershell
   Start-Process notepad.exe
   ```

   Found a corresponding **Event ID 1**, with a WindowsApps Notepad executable, command line, account, and parent-process information. A high-integrity indicator was observed, consistent with the previous elevated shell context, but is not by itself evidence of malicious activity.
10. Launched a second, more traceable parent-child test from a PowerShell session:

    ```powershell
    Start-Process cmd.exe -ArgumentList "/k echo SOC-LAB-TEST"
    ```

11. Found the matching **Event ID 1** for `C:\Windows\System32\cmd.exe` (observed PID **8680**) and verified these fields:
    - `CommandLine`: `"C:\WINDOWS\system32\cmd.exe" /k echo SOC-LAB-TEST`
    - `ParentImage`: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
    - `ParentCommandLine`: `"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"`

12. Inspected active Sysmon settings with `./Sysmon64.exe -c` in elevated PowerShell, confirming SHA256, no custom rules, and network connections disabled initially.
13. Created `sysmon-config.xml` to collect Event IDs 1 (process creation), 3 (network connection), 5 (process termination), and 22 (DNS query). Ran `./Sysmon64.exe -c ./sysmon-config.xml`; output confirmed: **Configuration file validated** and **Configuration updated**. Saved a copy under `config/sysmon-config.xml`.
14. Ran `Resolve-DnsName example.com` in the VM and confirmed a **Sysmon Event ID 22** with `QueryName: example.com`, `QueryStatus: 0`, and `Image: powershell.exe`.
15. Ran `Test-NetConnection -ComputerName example.com -Port 443`; observed `TcpTestSucceeded: True`, source IP `10.0.2.15` (VM NAT) and remote port 443. Checked **Sysmon Event ID 3**, verifying PowerShell-initiated TCP connection to the expected destination IP and port 443.
16. Shut down the VM and created a second snapshot `SOC-WIN11-Sysmon-Configured`. The VirtualBox screenshot shows the clean baseline snapshot and the post-Sysmon snapshot.

**Notes:** The initial `Test-NetConnection example.com -Port 443` command failed due to a parameter validation issue; specifying `-ComputerName` resolved it. The successful TCP test shows connectivity, not a verified HTTPS request or an attack. The broad Sysmon configuration is for local learning and needs tuning before ingesting data into Azure to avoid unnecessary log volume and cost.

## Investigation takeaway
A successful process-creation record links a child executable to its parent, launch arguments, time, and process identity. This benign experiment establishes that **Sysmon can capture the process relationship PowerShell → cmd.exe**, a useful prerequisite for later suspicious-process investigations. Neither this test nor the observed background events prove a security incident.

## Evidence (screenshots)
- `screenshots/02-sysmon/01-windows11-enterprise-desktop.png` — VM installation outcome.
- `screenshots/02-sysmon/02-clean-install-snapshot.png` — recovery snapshot.
- `screenshots/02-sysmon/03-extracted-sysmon-files.png` — Sysmon files prepared.
- `screenshots/02-sysmon/04-sysmon-operational-events.png` — Event Viewer operational channel with Sysmon events.
- `screenshots/02-sysmon/05-notepad-event1.png` — Notepad process event.
- `screenshots/02-sysmon/06-cmd-event1.png` — cmd.exe process-creation record.
- `screenshots/02-sysmon/07-cmd-commandline.png` — test command line.
- `screenshots/02-sysmon/08-cmd-parentimage.png` — PowerShell parent executable.
- `screenshots/02-sysmon/09-cmd-parentcommandline.png` — parent command line.
- `screenshots/02-sysmon/10-sysmon-configuration-updated.png` — validated/updated configuration output.
- `screenshots/02-sysmon/11-sysmon-event22-example-dns.png` — DNS query event (cropped/redacted).
- `screenshots/02-sysmon/12-network-test-succeeded.png` — successful TCP test result (cropped).
- `screenshots/02-sysmon/13-sysmon-event3-powershell.png` — PowerShell network event (redacted).
- `screenshots/02-sysmon/14-sysmon-event3-destination.png` — expected destination/port.
- `screenshots/02-sysmon/15-sysmon-configured-snapshot.png` — final two-snapshot VM state.

**Privacy:** Screenshots for this folder were selected or cropped to exclude visible personal account names, email addresses, and VM hostnames. Review again before any public GitHub push.

## Troubleshooting & lessons
- Entering a Windows path with spaces as if it were a PowerShell command produced `CommandNotFoundException`. The correct method was `cd "..."` followed by the executable command.
- Sysmon initially refused to install without administrator privileges; rerunning in elevated PowerShell resolved the problem.
- Seeing a Sysmon Event ID 1 for `conhost.exe` does not automatically identify the intended `cmd.exe` event; inspect `Image` and correlate `CommandLine` to select the relevant record.

## Next steps
- Start Phase 3 by verifying an Azure subscription and cost safeguards before creating cloud resources.
- Connect the Windows VM to Azure Arc / Azure Monitor Agent as appropriate, and selectively ingest the needed security events into Microsoft Sentinel.
- Reduce noisy Sysmon event volume and remove any personal information before public publishing.
