# Deployed SOC Lab Architecture — Verified October 24–25, 2026

```text
VirtualBox: SOC-WIN11-01 (Windows 11 Enterprise, NAT)
        |
  Sysmon service on Windows endpoint
        | Local log: Microsoft-Windows-Sysmon/Operational
        | Sysmon event IDs: 1 (process), 3 (network), 22 (DNS)
        v
 Azure Arc-connected machine + Azure Monitor Agent (AMA)
        |
  Data Collection Rule: dcr-soc-sysmon
  XPath: Microsoft-Windows-Sysmon/Operational!*[System[(EventID=1 or EventID=3 or EventID=22)]]
        v
 Log Analytics workspace: law-soc-lab-01
  Azure subscription: Azure for Students
  Resource group: rg-soc-lab
  Azure region: North Central US
  Daily ingestion cap: 0.05 GB/day
        v
 Microsoft Sentinel — KQL queries against Event table
        v
 Phase 4: detection engineering / Phase 5: completed incident investigations
```

## Verified
- Azure Arc showed **Connected** for the VM; Azure Monitor Agent extension reached **Succeeded**.
- Sysmon events ingested into the `Event` table and query returned **13 process events, 142 network connection events, 16 DNS queries** during one 24-hour query window.
- Windows Security Event Log exploration was a *separate Phase 1 activity*. Azure collection of Windows **Security** events was **not** configured during Phase 3; only specified Sysmon Operational events were forwarded.

## Scope / limitations
- VM uses NAT, not an isolated malware analysis network. Do not run untrusted malware or simulate attacks against third-party targets.
- Azure Arc connectivity/telemetry depends on the VM being powered on and connected to the internet.
- Azure budget is an alert only; Log Analytics daily cap is not a precise ceiling. Sentinel trial and student credits are time-limited.
- Azure cloud resources are not included in VirtualBox snapshots; the repository documents their names/configuration without exporting secrets or identifying account IDs.
