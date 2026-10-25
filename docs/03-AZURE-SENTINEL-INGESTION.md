# Phase 3 — Azure Arc, Log Analytics, Sentinel and Sysmon Ingestion

**Evidence date:** October 24–25, 2026 (based on completed screenshots and query output)

## Objective
Forward selected local Sysmon Windows Event Log records from a single owned Windows 11 VirtualBox VM to Microsoft Sentinel and demonstrate KQL visibility.

## Steps completed (verified)
1. Confirmed an Azure for Students subscription with $100 credit shown on the Education page at setup.
2. Created `SOC-Lab-Budget` ($5 per month, actual cost alerts at 50/80/100%). It is only a warning, not a spending cap.
3. Created Log Analytics workspace `law-soc-lab-01` in `rg-soc-lab`, **North Central US**. An attempt in East US was blocked by an Azure Policy allowed-regions restriction (`RequestDisallowedByAzure`); switching to an allowed region passed validation.
4. Configured workspace daily cap **0.05 GB/day**, confirmed by the Azure success toast. Ingestion overage and ancillary services can still incur charges; reaching the cap can interrupt visibility.
5. Enabled Sentinel and observed confirmation of a 31-day trial on the workspace (Oct 24–Nov 24, 2026, as shown by the portal).
6. Onboarded the Windows VM using Azure's generated Windows PowerShell script. Script execution was blocked by the PowerShell policy; `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force` applied **only in the current administrative session** and allowed the Microsoft-generated script to execute. Do not put tokens or onboarding script account identifiers in the repository.
7. Azure Arc Machines showed the endpoint **Connected**. The Azure Monitor Windows Agent extension was provisioned by data collection rule association and subsequently showed **Succeeded**.
8. Created DCR `dcr-soc-sysmon`, associated with the Arc machine and sending **Windows Event Logs** to `law-soc-lab-01`. The DCR used this custom XPath filter:

   ```text
   Microsoft-Windows-Sysmon/Operational!*[System[(EventID=1 or EventID=3 or EventID=22)]]
   ```

9. In Sentinel Logs, queried the `Event` table; real Sysmon events were visible. A 24-hour summary query returned:

   | Sysmon Event ID | Description | Count |
   |---|---|---:|
   | 1 | Process created | 13 |
   | 3 | Network connection detected | 142 |
   | 22 | DNS query | 16 |
   | **Total** | | **171** |

## Queries
See [`../queries/phase3-ingestion-validation.kql`](../queries/phase3-ingestion-validation.kql). Counts represent a point-in-time query window and will differ on subsequent runs.

## Evidence (sanitized)
- `../screenshots/03-sentinel/01-daily-cap-saved.png` — Azure confirmed the 0.05 GB/day setting.
- `../screenshots/03-sentinel/02-sentinel-trial-banner-cropped.png` — trial banner, cropped without account identifiers.
- `../screenshots/03-sentinel/03-dcr-basics.png` — DCR basics, without account IDs.
- `../screenshots/03-sentinel/04-dcr-sysmon-xpath.png` — selected Sysmon events.
- `../screenshots/03-sentinel/05-log-analytics-destination.png` — destination workspace selected.
- `../screenshots/03-sentinel/06-kql-event-summary.png` — final verified event counts.

## Troubleshooting / limitations
- Azure Policy regional restrictions can block otherwise valid resource deployment; inspect the subscription's *Allowed resource deployment regions* assignment and choose an approved Sentinel-supported region.
- PowerShell execution restrictions were resolved only for the current process, not permanently disabled.
- An Azure Arc asset showing as a resource is not necessarily *Connected*; confirm status, and separately verify the AMA extension succeeded.
- Agent success alone does not prove data ingestion. Verify with a KQL query and real results in the `Event` table.
- Do not confuse Azure Log Analytics **Event** table data (Sysmon channel) with Sentinel's separate **SecurityEvent** table. This phase did not configure general Windows Security Event log ingestion.
- The portal uses a subscription, resource group, and workspace name for configuration; omit tenant, subscription and workspace GUIDs, emails, tokens and onboarding scripts from public artifacts.
- The endpoint is a lab Windows client powered off between sessions, so near-real-time collection is only possible while it is running.
