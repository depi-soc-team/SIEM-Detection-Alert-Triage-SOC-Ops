# Data Source Inventory

## 1. Overview & Purpose
This document catalogs all monitored asset categories, log sources, event types, telemetry collection methods, and priority tiers for the SOC SIEM environment.

The scope covers both the **Enterprise Reference Environment (~300 users)** and the scaled **Hands-On Lab Environment**.

---

## 2. Monitored Data Source Matrix

| Source Category | Asset / Device | Operating System | Log Provider / Mechanism | Key Event Types Collected | Ingestion Method | SOC Value & Priority |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Identity / Directory** | Primary Domain Controller (`<PDC>`) | Windows Server 2022 | Active Directory DS, Security Event Log | 4624 (Logon), 4625 (Failed Logon), 4720 (User Created), 4728 (Group Member Added), 4672 (Admin Logon) | Elastic Agent (`windows` integration) | **P1 - Critical** |
| **Identity / Directory** | Additional Domain Controller (`<ADC>`) | Windows Server 2022 | Active Directory DS, Security Event Log | Replication logs, Kerberos Ticket Requests (4768, 4769), Account Lockouts (4740) | Elastic Agent (`windows` integration) | **P1 - Critical** |
| **Endpoints (Workstations)** | Workstations (`<PC-01>` to `<PC-04>`) and DCs (`<PDC>`, `<ADC>`) | Windows 10 / 11, Windows Server | Microsoft Sysmon v15+ | Event ID 1 (Process Create), Event ID 3 (Network Conn), Event ID 11 (File Create), Event ID 13 (Registry Set) | Elastic Agent (`windows` integration, Sysmon channel → `windows.sysmon_operational`) | **P1 - Critical** |
| **Endpoints (Workstations)** | User Workstations (`<PC-01>` to `<PC-04>`) | Windows 10 / 11 Enterprise | PowerShell Engine / ScriptBlock | Event ID 4104 (Script Block Execution), Event ID 4103 (Module Logging) via GPO | Elastic Agent (`windows` integration) | **P2 - High** |
| **Perimeter / Network** | FortiGate Firewall VM | FortiOS 7.x | FortiGate Traffic & UTM Syslog | Allowed/Dropped sessions, NAT translation, DNS queries, SSL inspection, IPS alerts | Syslog over UDP 9004 to the Elastic Agent (Fortinet FortiGate integration) on the SIEM node | **P1 - Critical** |
| **SIEM & Core Server** | SIEM All-in-One Node | Ubuntu 22.04 LTS | `auth.log`, `syslog` (auditd not collected) | SSH logins, `sudo` execution commands, service crashes, system changes | Elastic Agent (`system` integration) | **P2 - High** |
| **Cloud / Identity** *(Reference Design)* | Microsoft Entra ID / M365 | SaaS | Microsoft Graph API / M365 Management API | Sign-in logs, Conditional Access events, Risky users, Mailbox audit logs | Elastic API Integration | **P2 - High** |
| **Cloud Infrastructure** *(Reference Design)* | AWS / Azure Workloads | Cloud IaaS / PaaS | CloudTrail / Azure Activity Logs | IAM permission updates, Security group modifications, S3 bucket policy changes | Cloud Storage / EventHub Stream | **P3 - Medium** |

---

## 3. Telemetry Depth & Configuration Details

### 3.1 Endpoint Visibility (Sysmon + Elastic Agent)
* **Configuration Profile:** SwiftOnSecurity `sysmonconfig-export.xml` in the lab (see [log-onboarding-plan.md](log-onboarding-plan.md)); Olaf Hartong sysmon-modular is the alternative for wider coverage.
* **Coverage Scope:**
  * Process creation with full command line, parent process tree, and file hashes (SHA256).
  * Remote thread injection and process hollowing attempts (Event ID 8).
  * Suspicious outbound network connections initiated by native binaries (`certutil.exe`, `powershell.exe`, `rundll32.exe`).

### 3.2 Network Perimeter Visibility (FortiGate)
* **Log Types Forwarded:**
  * `type="traffic"`: All permitted and denied layer 3/4 network flows.
  * `type="utm"`: Antivirus, Web Filter, and Intrusion Prevention System (IPS) detections.
  * `type="event"`: Administrative logins and VPN tunnel establishments.

---

## 4. Priority Tiers Explained

1. **P1 (Critical):** Core authentication and process execution events. If collection stops, immediate blind spots occur. Retention: see below.
2. **P2 (High):** Scripting execution, command line logging, and host system logs. Essential for deep alert triage. Retention: see below.
3. **P3 (Medium):** High-volume informational flow logs, debug messages, and noisy application traces. Retention: see below.

**Retention:** the priority tiers set *monitoring importance*, not separate retention. All sources follow the single lifecycle in [retention-plan.md](retention-plan.md): 7 / 90 / 365 days in the reference design, and hot → delete after 90 days in the lab.

---

## 5. Lab Data Streams & MITRE ATT&CK Data Sources

The data streams actually ingesting in the lab, checked against Discover on `logs-*` (2026-10-02, see [02-onboarding/evidence.md](../02-onboarding/evidence.md)). Detection rules (#12) should be written against these datasets.

| Source | Data stream (`data_stream.dataset`) | Integration | Hosts | MITRE ATT&CK data sources | Owner (role) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Windows Security log | `system.security` | System | `<PDC>`, `<ADC>`, `<PC-01>`…`<PC-04>` | Logon Session (DS0028), User Account (DS0002), Active Directory (DS0026) | Infra & Onboarding |
| Sysmon | `windows.sysmon_operational` | Windows | `<PDC>`, `<ADC>`, `<PC-01>`…`<PC-04>` | Process (DS0009), Network Traffic (DS0029), File (DS0022), Windows Registry (DS0024) | Infra & Onboarding |
| PowerShell Operational | `windows.powershell_operational` | Windows | `<PDC>`, `<ADC>`, `<PC-01>`…`<PC-04>` | Script (DS0012), Command (DS0017) | Infra & Onboarding |
| Windows PowerShell (classic) | `windows.powershell` | Windows | Windows hosts | Command (DS0017) | Infra & Onboarding |
| Windows Defender | `windows.windows_defender` | Windows | Windows hosts | Process (DS0009), File (DS0022) | Infra & Onboarding |
| System / Application logs | `system.system`, `system.application` | System | Windows hosts | Service (DS0019) | Infra & Onboarding |
| FortiGate traffic | `fortinet_fortigate.log` | Fortinet FortiGate | SIEM node (UDP 9004 listener) | Network Traffic (DS0029), Firewall (DS0018) | Infra & Onboarding |
| Linux auth / syslog | `system.auth`, `system.syslog` | System | SIEM node | Logon Session (DS0028), Command (DS0017) | Infra & Onboarding |

`elastic_agent.*` datasets are agent self-monitoring and are excluded from security views (see [tuning notes](../03-detection-rules/tuning-notes.md)).

**Volume:** the whole lab produces roughly **1 EPS** on a normal day, on the order of 50,000–100,000 events per 24 h across all hosts on the SOC | Data Health dashboard. Per-source EPS is not measured yet. Use **Stack Management > Index Management > Data Streams** (document count ÷ age) to fill it in.
