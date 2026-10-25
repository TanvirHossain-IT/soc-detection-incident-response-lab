# Phase 5 — Controlled Simulations and Incident Triage

**Status:** Three automated incident case studies (one from Phase 4, two from Phase 5) completed; Rundll32 detection tuning tested and saved. Written case studies drafted for portfolio review.

- **User/network discovery:** `cmd.exe /c whoami` and `cmd.exe /c ipconfig` produced matching Sysmon 1 process records and **Incident #2**, closed as benign positive with analyst note.
- **Lab DNS pattern:** `Resolve-DnsName soc-test-001/002/003.example.com` produced three Sysmon 22 queries with status `9501`; a grouped KQL rule detected 3/3 distinct names and generated **Incident #3**, closed benign positive.
- **Rundll32:** two benign `shell32.dll,Control_RunDLL` invocations were captured through Sysmon 1. Tuned the query to exclude precisely the two controlled test invocations; result changed from a new test match to zero. No automated incident for Rundll32 was created.
- **Entity mapping:** host mapping initially absent from DNS Incident #3 despite detection. User corrected the rule afterward; revalidation on future alert not shown.

**Reports:** `reports/PHASE5-INCIDENT-RESPONSE-REPORT.md` and individual case studies.
**KQL:** `queries/phase5-*.kql`.
**Selected privacy-reviewed evidence:** `screenshots/05-incident-investigations/`.
