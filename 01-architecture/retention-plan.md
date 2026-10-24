# Data Retention & Index Lifecycle Management (ILM) Plan

## 1. Executive Summary & Purpose
Security operations produce continuous streams of high-volume log data. Retaining all raw telemetry indefinitely on primary storage is financially and architecturally unsustainable.

This **Data Retention Plan** balances three critical business drivers:
1. **Detection & Triage Capability:** Fast search performance for active investigations.
2. **Regulatory & Compliance Mandates:** Meeting requirements such as ISO/IEC 27001, PCI-DSS, and local cybersecurity regulations requiring 6 to 12 months of audit trails.
3. **Storage Cost Optimization:** Staging data through tiered storage via automated **Index Lifecycle Management (ILM)**.

---

## 2. Storage Tiers & ILM Phase Matrix

```
[ Ingest ] ──> [ Hot Tier ] ──(7 Days)──> [ Warm Tier ] ──(90 Days)──> [ Cold Tier ] ──(365 Days)──> [ Snapshot / Delete ]
                Fast NVMe                  High-Density               Object Storage /
                Read / Write               Read-Only                  S3 Glacier
```

| Lifecycle Tier | Age Range | Storage Media Type | Performance Profile | Shard Allocation & Configuration | Operations Permitted |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Hot** | Day 0 to 7 | High-speed NVMe / SSD | Maximum I/O, optimized for writing and real-time alert queries | 1 Primary, 1 Replica; Active indexing | Ingestion, Alert Queries, Dashboards, Live Hunting |
| **Warm** | Day 8 to 90 | Enterprise SATA SSD / SAS | Balanced cost, read-only search performance | Force-merged to 1 segment per shard; Read-only | Incident Root Cause Analysis, Threat Hunting, Compliance Audits |
| **Cold** | Day 91 to 365 | Dense HDD / Cloud Object Storage | Cost-optimized, slower query latency | Searchable snapshots; 0 replicas (backed by blob storage) | Historical breach audits, Legal hold requests |
| **Delete / Purge** | Day 366+ | — | Zero disk footprint | Automated index deletion following verified cold snapshot verification | Permanently deleted unless flagged for legal hold |

> **Age is counted from rollover, not ingest.** ILM `min_age` for warm/cold/delete is measured from the moment the index rolls over. With a hot rollover at `max_age: 7d`, a document can be up to ~14 days old when it reaches warm, and the other boundaries shift by up to 7 days too. The day ranges above are approximate.

---

## 3. Storage Sizing & Capacity Estimation

### 3.1 Enterprise Reference Organization (~300 Users)

| Telemetry Source Category | Events Per Second (EPS) | Average Event Size | Daily Volume (GB) | Hot Tier (7 Days) | Warm Tier (90 Days) | Annual Cold Volume (1 Year) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Windows Endpoints (300 PCs)** | 300 EPS | ~800 bytes | ~20.7 GB/day | ~145 GB | ~1.86 TB | ~7.5 TB |
| **Domain Controllers & Servers (15 VMs)**| 150 EPS | ~1.2 KB | ~15.5 GB/day | ~108 GB | ~1.40 TB | ~5.6 TB |
| **FortiGate Firewalls (Perimeter)** | 250 EPS | ~500 bytes | ~10.8 GB/day | ~75 GB | ~970 GB | ~3.9 TB |
| **Cloud Audit (M365 & Azure)** | 50 EPS | ~1.5 KB | ~6.5 GB/day | ~45 GB | ~585 GB | ~2.4 TB |
| **Total Estimated Footprint** | **~750 EPS** | — | **~53.5 GB/day** | **~373 GB** | **~4.81 TB** | **~19.4 TB** |

*Note: Elasticsearch native LZ4/Deflate compression typically reduces raw JSON volume by 35% to 50% upon index rollover.*

### 3.2 Hands-On Lab Sizing (4 Workstations + 2 DCs + 1 FortiGate)
* **Observed Lab Throughput:** roughly **1 EPS** on a normal day. The SOC | Data Health dashboard on 2026-10-02 showed on the order of 50,000–100,000 events per 24 h across all hosts (see [04-dashboards](../04-dashboards/README.md)).
* **Peak / planning figure:** 10–25 EPS is kept as an upper bound for attack-simulation days.
* **Daily Volume:** at ~1 KB per event, about **0.05–0.1 GB/day** observed, up to ~1–2 GB/day at the peak figure. Confirm with real data stream sizes in **Stack Management > Index Management > Data Streams**.
* **Lab Disk Allocation:** a 50 GB virtual disk on the single Ubuntu SIEM node comfortably holds the full project period (October to December) without tiered offloading.

---

## 4. Elasticsearch ILM Policy Definition

### 4.1 Reference (production) policy: `soc-ilm-policy`

The policy below enforces the reference lifecycle above. It is the **production design for the reference organization and is not deployed in the lab**:

* The cold phase uses `searchable_snapshot`, which needs an **Enterprise** license (verify in **Stack Management > License Management**) and a registered snapshot repository (`soc_backup_repo`). The lab has neither.
* Warm/cold tiers need dedicated data-tier nodes. The lab runs a single Elasticsearch node.

```json
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "7d"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "forcemerge": {
            "max_num_segments": 1
          },
          "shrink": {
            "number_of_shards": 1
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "90d",
        "actions": {
          "searchable_snapshot": {
            "snapshot_repository": "soc_backup_repo"
          },
          "set_priority": {
            "priority": 20
          }
        }
      },
      "delete": {
        "min_age": "365d",
        "actions": {
          "delete": {
            "delete_searchable_snapshot": true
          }
        }
      }
    }
  }
}
```

### 4.2 Lab policy: `soc-lab-ilm-policy` (proposed)

The single-node lab uses a **hot → delete** lifecycle only. 90 days keeps every log from the start of onboarding (2026-10-02) available until the final presentation (2026-12-04).

```json
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "10gb",
            "max_age": "7d"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

**How to apply:** Fleet-managed integration data streams use the built-in `logs` ILM policy by default. To switch a data stream to `soc-lab-ilm-policy`, set `index.lifecycle.name` in that data stream's `@custom` component template (e.g. `logs-system.security@custom`, `logs-windows.sysmon_operational@custom`, `logs-fortinet_fortigate.log@custom`). The change applies from the next rollover.

**Status:** proposed, not yet applied. Issue #3 still needs the screenshot showing the policy attached to the data streams (**Stack Management > Index Lifecycle Policies > soc-lab-ilm-policy > linked indices**).
