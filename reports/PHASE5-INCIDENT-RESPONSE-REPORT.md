# Phase 5 — Detection and Incident Response Report

**Environment:** Windows 11 Enterprise lab VM in VirtualBox; Sysmon; Azure Arc; Azure Monitor Agent; Microsoft Sentinel; KQL. **All activities were authorized, non-destructive lab tests.**

## Executive summary

Developed and tested detection content using Sysmon Events 1 and 22, validated scheduled alerts and incidents, conducted manual triage, and closed three test-driven incidents as benign positives. The first encoded PowerShell incident was created in Phase 4; Windows reconnaissance and DNS incidents were created in Phase 5. A fourth case study investigated benign `rundll32.exe` invocations and illustrated how to tune a hunting query against known lab false positives. No malware, compromise, persistence, credential theft, or exfiltration was observed or demonstrated.

## Case register

| Case | Source event | Finding | Alert / incident | Disposition |
|---|---|---|---|---|
| Encoded PowerShell | Sysmon 1 | `-EncodedCommand` used to print harmless lab text | Sentinel Incident #1, Medium | Benign positive; closed |
| User and network discovery | Sysmon 1 | `cmd.exe` → `whoami.exe` and `ipconfig.exe` | Sentinel Incident #2, Low | Benign positive; closed |
| Multiple unsuccessful DNS lookups | Sysmon 22 | Three failed queries to controlled `soc-test-*.example.com` hosts, launched by PowerShell | Sentinel Incident #3, Low | Benign positive; closed |
| Rundll32 execution / tuning | Sysmon 1 | Two Windows Control Panel invocations; query tuned to exclude only these tests | Manual hunt only; no incident | Benign observed activity |

## Workflow and evidence

1. Executed safe commands in the isolated-to-purpose Windows test VM (NAT connectivity is not complete isolation).
2. Confirmed process creation / DNS events with Microsoft Sysmon, forwarded through Azure Arc and AMA into Sentinel `Event` logs.
3. Inspected timestamps, process images, command lines, DNS names, parent process context and host attribution.
4. Wrote and tested KQL queries; created scheduled Sentinel analytics rules for encoded PowerShell, reconnaissance and test DNS-lookup patterns.
5. Validated generated alerts and incident records for cases 1–3, then added documented benign-positive closures.
6. Tuned a separate Rundll32 hunting query with controlled exclusions and compared results before and after tuning.

## ATT&CK context

- **T1059.001 — PowerShell:** encoded execution pattern; benign test.
- **T1033 — System Owner/User Discovery:** `whoami`.
- **T1016 — System Network Configuration Discovery:** `ipconfig`.
- **T1218.011 — Rundll32:** dual-use Windows utility; use in this lab was not malicious or evasive.
- The lab-specific DNS lookup detector has **no asserted ATT&CK technique mapping**; it does not demonstrate DNS tunneling/exfiltration.

## Limitations and next steps

- These demonstrations verify detection logic on controlled input, **not accuracy, coverage, or efficacy against real adversaries**.
- Sentinel scheduled alerts are delayed relative to event time and can be affected by lookup and suppression settings. Lab rules used a **5-minute run frequency**, **15-minute lookback**, a **greater-than-zero** match threshold, grouped alerts and **20-minute suppression**; suppressing a rule pauses further evaluations during that period.
- The DNS incident initially showed zero mapped entities; entity mapping was corrected afterward but has **not been validated using a new alert**.
- Some queries rely on regex parsing of `RenderedDescription` and are dependent on the collected message format. Production detections should use structured fields when available, refine process path matching, consider ingestion latency, and avoid overly broad exclusions.
- The DNS and reconnaissance rules are intentionally narrow or noisy lab demonstrations, not production-quality detections.
- Do not publish private Azure account details, IDs, student email addresses, local usernames, or unredacted security logs. Curated screenshots in this archive were selected to minimize personal identifiers; still review all images before a public commit.
- Confirm current Azure costs; remove or disable monitoring resources and automated rules when no longer needed.

## Full case studies

- [Incident 1 — Encoded PowerShell](01-encoded-powershell.md)
- [Incident 2 — Windows reconnaissance](02-windows-reconnaissance.md)
- [Incident 3 — DNS lookup pattern](03-dns-lookup-pattern.md)
- [Case 4 — Rundll32 tuning](04-rundll32-tuning.md)
