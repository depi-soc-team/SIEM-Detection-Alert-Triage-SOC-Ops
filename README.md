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

## 4-Week Plan

| Week | Focus | Key Deliverables |
|------|-------|------------------|
| 1 | Architecture & Onboarding | Lab architecture design, Elastic Stack deployment, Fleet + agents enrolled, first log sources ingesting |
| 2 | Detection Engineering | Custom detection rules with ATT&CK mapping, attack simulation scenarios, rule validation |
| 3 | Triage & Dashboards | Alert triage reports, runbooks, SOC dashboards and KPIs, rule tuning |
| 4 | SOAR, AI & Final Report | Automation playbooks, AI-assisted triage, final report, presentation and demo |

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
