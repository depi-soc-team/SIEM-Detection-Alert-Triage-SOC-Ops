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
* **Average Lab Throughput:** 10–25 EPS.
* **Daily Volume:** ~1.2 GB/day.
* **Lab Disk Allocation:** 50 GB virtual disk on single Ubuntu SIEM node accommodates over 30 days of uncompressed lab testing without requiring tiered offloading.

---

## 4. Elasticsearch ILM Policy Definition

Below is the automated JSON policy deployed to Elasticsearch (`soc-ilm-policy`) that enforces this lifecycle:

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
