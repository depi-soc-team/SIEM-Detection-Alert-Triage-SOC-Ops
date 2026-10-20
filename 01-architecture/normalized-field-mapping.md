# Normalized Field Mapping (Elastic Common Schema - ECS)

## 1. What is Field Normalization & Why is it Critical?
Raw security logs originate from dozens of different vendors, each using disparate terminology for the exact same concept:
* Windows logs identity as `TargetUserName`.
* Linux logs identity as `USER` or `pam_user`.
* Firewalls log identity as `src_user` or `usrName`.

Without normalization, a security analyst or automated detection rule would need to write separate queries for every vendor:
```text
# WITHOUT Normalization (Painful & error-prone):
TargetUserName: "admin" OR USER: "admin" OR src_user: "admin" OR usrName: "admin"
```

**With Field Normalization (Elastic Common Schema - ECS):**
Every single log source is parsed into unified standard fields:
```text
# WITH Normalization:
user.name: "admin"
```
A single detection rule or dashboard panel immediately works across all operating systems, firewalls, and cloud platforms simultaneously.

---

## 2. ECS Core Field Mappings

### 2.1 Identity & Authentication (Windows Security Event Log)

| Raw Windows Field (Event ID 4624 / 4625) | ECS Standard Field | Field Data Type | Description |
| :--- | :--- | :--- | :--- |
| `TargetUserName` | `user.name` | `keyword` | Name of the authenticating user |
| `TargetDomainName` | `user.domain` | `keyword` | Domain of the user account |
| `TargetUserSid` | `user.id` | `keyword` | Unique Security Identifier (SID) |
| `IpAddress` | `source.ip` | `ip` | Origin IP of the logon request |
| `IpPort` | `source.port` | `long` | Origin source port |
| `LogonType` | `winlog.logon.type` / `event.action` | `keyword` | Logon method (e.g. 2=Interactive, 3=Network, 10=RDP) |
| `Status` / `SubStatus` | `event.outcome` | `keyword` | `success` (4624) or `failure` (4625) |

---

### 2.2 Endpoint Process Telemetry (Sysmon Event ID 1)

| Raw Sysmon Field (Event ID 1) | ECS Standard Field | Field Data Type | Description |
| :--- | :--- | :--- | :--- |
| `UtcTime` | `@timestamp` | `date` | Standardized UTC execution time |
| `Image` | `process.executable` | `keyword` | Full path to the executable |
| `CommandLine` | `process.command_line` | `wildcard` / `text` | Complete arguments and switches passed |
| `ProcessId` | `process.pid` | `long` | Operating system process identifier |
| `ParentImage` | `process.parent.executable` | `keyword` | Executable that spawned this process |
| `ParentCommandLine` | `process.parent.command_line`| `wildcard` | Full command line of parent process |
| `ParentProcessId` | `process.parent.pid` | `long` | Process ID of the parent |
| `User` | `user.name` | `keyword` | Security context the process ran under |
| `Hashes` (SHA256) | `process.hash.sha256` | `keyword` | Extracted cryptographic file hash |
| `CurrentDirectory` | `process.working_directory` | `keyword` | Working path where execution started |

---

### 2.3 Network Traffic (FortiGate Firewall Syslog)

| Raw FortiGate Syslog Key | ECS Standard Field | Field Data Type | Description |
| :--- | :--- | :--- | :--- |
| `srcip` | `source.ip` | `ip` | Originating host IP address |
| `srcport` | `source.port` | `long` | Source port number |
| `dstip` | `destination.ip` | `ip` | Destination server / host IP address |
| `dstport` | `destination.port` | `long` | Target service port (e.g. 80, 443, 53) |
| `proto` (e.g. `6` / `17`) | `network.transport` | `keyword` | Transport protocol (`tcp`, `udp`, `icmp`) |
| `action` (`accept` / `deny` / `close`)| `event.action` / `event.outcome` | `keyword` | `allowed` or `denied` |
| `policyid` | `rule.id` | `keyword` | Firewall security policy rule index |
| `sentbyte` | `source.bytes` | `long` | Bytes transmitted from source |
| `rcvdbyte` | `destination.bytes` | `long` | Bytes returned from destination |
| `duration` | `event.duration` | `long` | Session duration (nanoseconds/seconds) |

---

## 3. Normalization Ingest Pipeline Example

Below is the conceptual Elasticsearch Ingest Pipeline definition that normalizes raw FortiGate syslog into ECS:

```json
{
  "description": "Normalize FortiGate Syslog to Elastic Common Schema",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": [
          "srcip=%{IP:source.ip} srcport=%{INT:source.port:long} dstip=%{IP:destination.ip} dstport=%{INT:destination.port:long} action=%{WORD:temp_action} proto=%{INT:network.iana_number}"
        ]
      }
    },
    {
      "set": {
        "field": "event.category",
        "value": "network"
      }
    },
    {
      "script": {
        "lang": "painless",
        "source": "if (ctx.temp_action == 'accept' || ctx.temp_action == 'pass') { ctx.event.outcome = 'success'; ctx.event.action = 'allowed'; } else { ctx.event.outcome = 'failure'; ctx.event.action = 'denied'; }"
      }
    },
    {
      "remove": {
        "field": "temp_action"
      }
    }
  ]
}
```
