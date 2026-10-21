# Data Source Inventory

## 1. Overview & Purpose
This document catalogs all monitored asset categories, log sources, event types, telemetry collection methods, and priority tiers for the SOC SIEM environment.

The scope covers both the **Enterprise Reference Environment (~300 users)** and the scaled **Hands-On Lab Environment**.

---

## 2. Monitored Data Source Matrix

| Source Category | Asset / Device | Operating System | Log Provider / Mechanism | Key Event Types Collected | Ingestion Method | SOC Value & Priority |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Identity / Directory** | Primary Domain Controller (`<PDC>`) | Windows Server 2022 | Active Directory DS, Security Event Log | 4624 (Logon), 4625 (Failed Logon), 4720 (User Created), 4728 (Group Member Added), 4672 (Admin Logon) | Elastic Agent (`windows` integration) | **P1 - Critical** |
| **Identity / Directory** | Backup Domain Controller (`<ADC>`) | Windows Server 2022 | Active Directory DS, Security Event Log | Replication logs, Kerberos Ticket Requests (4768, 4769), Account Lockouts (4740) | Elastic Agent (`windows` integration) | **P1 - Critical** |
| **Endpoints (Workstations)** | User Workstations (`<PC-01>` to `<PC-04>`) | Windows 10 / 11 Enterprise | Microsoft Sysmon v15+ | Event ID 1 (Process Create), Event ID 3 (Network Conn), Event ID 11 (File Create), Event ID 13 (Registry Set) | Elastic Agent (`system` + `sysmon` integration) | **P1 - Critical** |
| **Endpoints (Workstations)** | User Workstations (`<PC-01>` to `<PC-04>`) | Windows 10 / 11 Enterprise | PowerShell Engine / ScriptBlock | Event ID 4104 (Script Block Execution), Event ID 4103 (Module Logging) via GPO | Elastic Agent (`windows` integration) | **P2 - High** |
| **Perimeter / Network** | FortiGate Firewall VM | FortiOS 7.x | FortiGate Traffic & UTM Syslog | Allowed/Dropped sessions, NAT translation, DNS queries, SSL inspection, IPS alerts | Syslog over UDP port 9004 to SIEM Forwarder | **P1 - Critical** |
| **SIEM & Core Server** | SIEM All-in-One Node | Ubuntu 22.04 LTS | Linux `auditd`, `auth.log`, `syslog` | SSH logins, `sudo` execution commands, service crashes, system changes | Elastic Agent (`system` integration) | **P2 - High** |
| **Cloud / Identity** *(Reference Design)* | Microsoft Entra ID / M365 | SaaS | Microsoft Graph API / M365 Management API | Sign-in logs, Conditional Access events, Risky users, Mailbox audit logs | Elastic API Integration | **P2 - High** |
| **Cloud Infrastructure** *(Reference Design)* | AWS / Azure Workloads | Cloud IaaS / PaaS | CloudTrail / Azure Activity Logs | IAM permission updates, Security group modifications, S3 bucket policy changes | Cloud Storage / EventHub Stream | **P3 - Medium** |

---

## 3. Telemetry Depth & Configuration Details

### 3.1 Endpoint Visibility (Sysmon + Windows Event Forwarding)
* **Configuration Profile:** SwiftOnSecurity / Olaf Hartong Sysmon modular XML configuration.
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

1. **P1 (Critical):** Core authentication and process execution events. If collection stops, immediate blind spots occur. Retention: 90 days active, 365 days cold.
2. **P2 (High):** Scripting execution, command line logging, and host system logs. Essential for deep alert triage. Retention: 30 days active, 180 days cold.
3. **P3 (Medium):** High-volume informational flow logs, debug messages, and noisy application traces. Retention: 7 to 14 days active.
