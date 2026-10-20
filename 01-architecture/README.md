# 01 - SIEM Architecture & Log Onboarding (Week 1 Deliverables)

This directory contains the complete **Week 1 Deliverables** for the SIEM Detection Engineering, Alert Triage & SOC Operations project.

---

## Deliverables Index

| # | Deliverable | Description | Link |
|---|:---|:---|:---|
| 1 | **SIEM Architecture** | Complete system design, reference model (~300 users), lab implementation, and data flow. | [siem-architecture.md](siem-architecture.md) |
| 2 | **Architecture Diagrams** | Draw.io vector sources and rendered high-resolution PNGs. | [Reference PNG](siem-reference-architecture.png) · [Lab PNG](siem-lab-architecture.png) |
| 3 | **Data Source Inventory** | Catalog of all monitored hosts, event IDs, network sources, and priority tiers. | [data-source-inventory.md](data-source-inventory.md) |
| 4 | **Log Onboarding Plan** | Technical procedures for Sysmon, Windows Events, FortiGate syslog, and verification queries. | [log-onboarding-plan.md](log-onboarding-plan.md) |
| 5 | **Normalized Field Mapping** | Cross-vendor field mapping into the Elastic Common Schema (ECS) with sample ingest pipeline. | [normalized-field-mapping.md](normalized-field-mapping.md) |
| 6 | **Data Retention Plan** | Hot/Warm/Cold ILM storage lifecycle policy, capacity estimation, and compliance strategy. | [retention-plan.md](retention-plan.md) |
| 7 | **Architecture Decision Records** | Key design decisions (e.g. Tailscale secure remote access). | [ADR-001](adr/ADR-001-remote-access-tailscale.md) |

---
**Role Responsible:** Lead & SIEM Architect
