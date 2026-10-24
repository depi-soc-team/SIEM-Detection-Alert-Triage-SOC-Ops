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
1. Assign the agent policy used in the lab:
   * Workstations (`<PC-01>` to `<PC-04>`): `Windows Policy`
   * Domain controllers (`<PDC>`, `<ADC>`): `Windows Server Policy`
   * SIEM node: `Fleet Server Policy`
2. Execute enrollment script on target Windows host:
```powershell
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri "https://<FLEET_SERVER_IP>:8220/api/status" -SkipCertificateCheck
.\elastic-agent.exe install --url="https://<FLEET_SERVER_IP>:8220" --enrollment-token="<ENROLLMENT_TOKEN>" --insecure
```
> `-SkipCertificateCheck` needs PowerShell 7+ (Windows PowerShell 5.1 does not support it). `--insecure` skips TLS verification and is only acceptable in the lab with the self-signed Fleet Server certificate; in production use `--certificate-authorities=<CA_PATH>` instead. Never commit the real enrollment token.

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
In Fleet -> Integrations -> **Fortinet FortiGate Firewall Logs** (package `fortinet_fortigate`), added to `Fleet Server Policy` on the SIEM node:
* Input type: `UDP`
* Listen address: `0.0.0.0:9004`
* Data stream: `logs-fortinet_fortigate.log-default` (dataset `fortinet_fortigate.log`)
* Ingest pipeline: managed by the integration (no custom pipeline needed)
* Kibana data view: `SOC - FortiGate` = `logs-fortinet_fortigate.*` (see [02-onboarding/data-views.md](../02-onboarding/data-views.md))

---

## 4. Log Onboarding Verification & Evidence

To prove onboarding was successful, analysts execute verification checks across the SIEM stack.

### Verification Checklist:
Screenshots: [02-onboarding/evidence.md](../02-onboarding/evidence.md).

* [x] **Agent Status**: Fleet displays all 6 agents as `Healthy` with steady heartbeats.
* [x] **Stream Discovery**: Events are visible in Kibana Discover under data view `SOC - Windows` (`logs-windows*,logs-system*`).
* [x] **Network Stream Discovery**: Firewall events visible under data view `SOC - FortiGate` (`logs-fortinet_fortigate.*`).
* [ ] **Timestamp Check**: `@timestamp` matches real-world UTC generation time within $\pm 2$ seconds. *(not yet evidenced)*
* [ ] **No Unparsed Errors**: Field `error.message` is absent in indexed documents. *(not yet evidenced)*

> Data view patterns: use `logs-windows*`, not `logs-windows.*` (the latter did not match in the data view wizard). `host.name` is stored lowercase and KQL values are case-sensitive.

### Verification Query Examples (KQL - Kibana Query Language):
```text
1. Verify Windows Security logon events (data view: SOC - Windows)
   event.code : "4624" and data_stream.dataset : "system.security"

2. Verify Sysmon process creation telemetry (data view: SOC - Windows)
   event.code : "1" and data_stream.dataset : "windows.sysmon_operational"

3. Verify FortiGate network traffic logs (data view: SOC - FortiGate)
   data_stream.dataset : "fortinet_fortigate.log" and network.transport : "tcp"

4. Check for parsing errors (data view: logs-*)
   error.message : *
```
