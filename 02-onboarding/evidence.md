# Log Onboarding Evidence

Screenshots proving each component and log source works. Captured 2026-10-02. Sanitized: real IPs and usernames are redacted, and lab hostnames are kept.

> Unredacted originals are kept in the team Google Drive (`01-Evidence/Phase2-Design-LabSetup`), not in the repo.

| Image | Caption | Supports |
|-------|---------|----------|
| [data-sources-inventory](images/20261002_basel_data-sources-inventory.png) | Discover on `logs-*`: datasets ingesting in the last 24 h (Sysmon, syslog, Windows Security, FortiGate, system, agent) | #2, #6 |
| [fleet-agents-healthy](images/20261002_basel_fleet-agents-healthy.png) | Fleet: 6 agents Healthy across Windows Policy, Windows Server Policy and Fleet Server Policy | #4 |
| [fortigate-logs-ingested](images/20261002_basel_fortigate-logs-ingested.png) | FortiGate syslog arriving in `fortinet_fortigate.log` via the Elastic Agent on the SIEM host (IP redacted) | #5, #6 |
| [fortigate-forward-traffic](images/20261002_basel_fortigate-forward-traffic.png) | FortiGate forward traffic in Discover via the Fortinet integration (IPs redacted) | #5, #6 |
| [powershell-4104-test](images/20261002_basel_powershell-4104-test.png) | Test script block `SOC-TEST-4104` captured as event 4104 in `windows.powershell_operational` | #6 |
| [sysmon-ingested-pdc](images/20261002_basel_sysmon-ingested-pdc.png) | Sysmon process-creation events (event code 1) from the domain controller `<PDC>` in `windows.sysmon_operational` | #6 |
| [vm-images](images/20261002_basel_vm-images.png) | VMware inventory of lab VMs (Windows clients, Windows Server, Elastic, FortiGate, Kali, Metasploitable2) | #8 |

Dashboard screenshots: [04-dashboards/README.md](../04-dashboards/README.md).
