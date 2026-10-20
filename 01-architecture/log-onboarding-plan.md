# Log Onboarding Plan & Verification Evidence

## 1. Onboarding Strategy
This document defines the technical procedure for onboarding new log sources into the Elastic SIEM deployment, validating ingestion integrity, and verifying that data fields match the Elastic Common Schema (ECS).

---

## 2. Onboarding Workflow

The standardized 5-step onboarding lifecycle:

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  1. Requirements │ ──> │  2. Agent/Syslog │ ──> │  3. Parsing &    │ ──> │  4. Ingestion    │ ──> │  5. Sign-off &   │
│   Identification │     │   Configuration  │     │   Normalization  │     │    Verification  │     │   Rule Mapping   │
└──────────────────┘     └──────────────────┘     └──────────────────┘     └──────────────────┘     └──────────────────┘
```

1. **Requirements Identification**: Determine what events the data source produces and what MITRE ATT&CK techniques it covers.
2. **Agent / Shipper Configuration**: Deploy Elastic Agent via Fleet Server policy or configure network device syslog forwarding.
3. **Parsing & Ingest Pipeline**: Direct raw data into the appropriate Elasticsearch ingest pipeline for grok parsing and field extraction.
4. **Ingestion Verification**: Validate that raw event count is greater than zero and events are timestamped accurately in UTC.
5. **Sign-off & Rule Mapping**: Confirm data view indexing (`logs-*-*`) and assign detection rules.

---

## 3. Step-by-Step Onboarding Procedures

### 3.1 Windows Workstations & Domain Controllers (`<PDC>`, `<ADC>`, `<PC-01>` to `<PC-04>`)

#### Step 1: Install Sysmon
Run an elevated PowerShell session to install Sysmon with standard configuration:
```powershell
Invoke-WebRequest -Uri "https://live.sysinternals.com/Sysmon64.exe" -OutFile "C:\Windows\Temp\Sysmon64.exe"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Windows\Temp\sysmonconfig.xml"
C:\Windows\Temp\Sysmon64.exe -accepteula -i C:\Windows\Temp\sysmonconfig.xml
```

#### Step 2: Deploy Elastic Agent via Fleet
In Kibana -> **Management -> Fleet -> Add Agent**:
1. Assign agent policy: `Windows-Workstation-Policy` or `Domain-Controller-Policy`.
2. Execute enrollment script on target Windows host:
```powershell
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri "https://<FLEET_SERVER_IP>:8220/api/status" -SkipCertificateCheck
.\elastic-agent.exe install --url="https://<FLEET_SERVER_IP>:8220" --enrollment-token="<ENROLLMENT_TOKEN>" --insecure
```

#### Step 3: Enable GPO for PowerShell Audit
On the Primary Domain Controller (`<PDC>`), apply GPO `SOC-Logging-Policy`:
* `Computer Configuration -> Administrative Templates -> Windows Components -> Windows PowerShell`:
  * **Turn on PowerShell Script Block Logging**: Enabled (Logs Event ID 4104).
  * **Turn on PowerShell Module Logging**: Enabled with module name `*`.

---

### 3.2 Perimeter Network Logs (FortiGate VM)

#### Step 1: Configure Syslog Destination on FortiGate CLI
Point syslog output to the SIEM Ubuntu server on dedicated UDP port 9004:
```text
config log syslogd setting
    set status enable
    set server "<SIEM_UBUNTU_IP>"
    set mode udp
    set port 9004
    set facility local7
    set format default
end
```

#### Step 2: Configure Elastic Agent Syslog Listener
In Fleet -> Integrations -> **Custom Logs / Fortinet Integration**:
* Input type: `UDP syslog`
* Listen address: `0.0.0.0:9004`
* Index pattern: `logs-fortinet.firewall-*`
* Ingest pipeline: `logs-fortinet.firewall-pipeline`

---

## 4. Log Onboarding Verification & Evidence

To prove onboarding was successful, analysts execute verification checks across the SIEM stack.

### Verification Checklist:
* [x] **Agent Status**: Fleet displays agent status as `Healthy` with steady heartbeats.
* [x] **Stream Discovery**: Events are visible in Kibana Discover under data view `SOC - Windows` (`logs-windows.*-*`).
* [x] **Network Stream Discovery**: Firewall events visible under data view `SOC - FortiGate` (`logs-fortinet.*-*`).
* [x] **Timestamp Check**: `@timestamp` matches real-world UTC generation time within $\pm 2$ seconds.
* [x] **No Unparsed Errors**: Field `error.message` is absent in indexed documents.

### Verification Query Examples (KQL - Kibana Query Language):
```kql
# 1. Verify Windows Security Logon Events
event.code: 4624 and winlog.channel: "Security"

# 2. Verify Sysmon Process Creation Telemetry
event.code: 1 and winlog.channel: "Microsoft-Windows-Sysmon/Operational"

# 3. Verify FortiGate Network Traffic logs
event.dataset: "fortinet.firewall" and network.transport: "tcp"
```
