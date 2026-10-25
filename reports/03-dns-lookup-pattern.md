# Incident 3 — Multiple unsuccessful DNS lookups (benign positive)

**Scenario:** Three authorized DNS lookup attempts to reserved/example test hostnames.

- **Commands:** `Resolve-DnsName` for `soc-test-001.example.com`, `soc-test-002.example.com`, and `soc-test-003.example.com`.
- **Source:** Sysmon Event ID 22, process `powershell.exe`.
- **Observed:** Three DNS events with `QueryStatus` **9501**, around **2026-10-25 03:23:53 UTC**. The test hostnames did not resolve normally; this is expected for the exercise.
- **Detection:** KQL rule correlated **3 failed lookups** across **3 distinct names**, grouped by machine and process, producing **one detection result**.
- **Alert:** Scheduled rule `SOC - Suspicious DNS Activity Detection`, Low severity; incident #3 opened automatically with one alert at approximately **2026-10-25 03:30 UTC**.
- **Triage:** Verified the query results, responsible executable, and intentional test procedure.
- **Disposition:** Incident #3 closed **Benign Positive — Suspicious but expected** with closure explanation.
- **Caveat:** The incident initially displayed **Entities (0)** even though a host name appeared elsewhere. The host entity mapping was subsequently corrected, but we have not independently re-tested entity enrichment on a newer DNS alert. Do not claim it was verified after correction.
- **Lesson:** Multiple failed DNS requests can be a useful signal for triage, but do **not** establish DNS tunneling, exfiltration, or compromise.
