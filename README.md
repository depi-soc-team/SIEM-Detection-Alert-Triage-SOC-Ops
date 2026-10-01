# SIEM Detection Engineering, Alert Triage & SOC Operations

DEPI graduation project built on the **Elastic Stack** (Elasticsearch, Kibana, Fleet / Elastic Agent, Elastic Security).

## Goal

Design and operate a small, realistic Security Operations Center (SOC) lab that:

- Collects and normalizes security telemetry from endpoints and network sources
- Detects adversary behavior with custom, MITRE ATT&CK–mapped detection rules
- Validates detections through controlled attack simulation
- Triages alerts using a consistent, documented process
- Visualizes SOC health and threat activity in Kibana dashboards
- Automates enrichment and response with SOAR and AI-assisted workflows

## Project Timeline

| Phase | Milestone | Due | Key Deliverables |
|-------|-----------|-----|------------------|
| 1 | M1 - Planning, Literature Review & Requirements | 2026-10-16 | Project plan, literature review, requirements specification, team & roles reported to instructor |
| 2 | M2 - System Analysis & Design + Lab Setup | 2026-11-06 | SIEM architecture, data source inventory, retention/ILM, Fleet + Sysmon, FortiGate + VPN, onboarding evidence, ECS field mapping, lab VMs, Kibana RBAC, detection use cases, KPI definitions |
| 3 | M3 - Implementation | 2026-11-30 | Detection rules, attack simulations, SOC dashboard, triage reports, TI enrichment, escalation templates, ATT&CK coverage map, Shuffle SOAR playbook, AI-assisted triage |
| 4 | M4 - Testing, Reports & Final Presentation | 2026-12-04 | Test results, rule tuning, final report & executive summary, presentation & demo |

## Team

| Member | Role |
|--------|------|
| Basel Mostafa | Lead & Architect |
| Merna Walid | Infra & Onboarding |
| Ramez Karam | Detection Engineering |
| Saieed Mohamed | Attack Simulation & SOC Analyst |
| Ahmed El-Najjar | Dashboards, SOAR & AI |

## Repository Structure

```
01-architecture/      Architecture diagrams and design decisions
02-onboarding/        Stack setup and log source onboarding
03-detection-rules/   Detection rules (see rule-template.md)
04-dashboards/        Kibana dashboards and KPIs
05-triage/            Triage reports and runbooks (see triage-template.md)
06-soar-ai/           SOAR playbooks and AI-assisted workflows
final-report/         Final deliverables
```

## Security Note

This repository must not contain real IP addresses, hostnames, usernames, credentials, tokens, or raw log/packet captures. Use placeholders (e.g. `<DC01>`, `10.0.0.x`, `<user>`) and keep secrets in a git-ignored `.env` file.
