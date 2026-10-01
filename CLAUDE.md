# Project Conventions

DEPI SOC project: SIEM detection engineering, alert triage, and SOC operations on the Elastic Stack. This repo holds documentation, configs, rules, dashboards, and reports — not a running application.

## Structure

- `01-architecture/` — design docs and diagrams
- `02-onboarding/` — stack setup and log source integration
- `03-detection-rules/` — one rule per file, based on `rule-template.md`
- `04-dashboards/` — Kibana saved-object exports and dashboard docs
- `05-triage/` — triage reports and runbooks, based on `triage-template.md`
- `06-soar-ai/` — automation playbooks and AI-assisted workflows
- `final-report/` — final deliverables

Each folder's `README.md` describes what belongs there.

## Security Rules (mandatory)

- Never write real IP addresses, hostnames, domains, usernames, passwords, API keys, enrollment tokens, or certificates into any file.
- Use placeholders: `<host-01>`, `<DC01>`, `<user>`, `10.0.0.x`, `example.local`, `<ELASTIC_PASSWORD>`.
- Secrets go in `.env` (git-ignored) and are referenced by variable name only.
- Do not commit raw logs or captures (`*.evtx`, `*.pcap`); commit sanitized excerpts instead.
- Defang IOCs in reports (`hxxp://`, `1.2.3[.]4`).

## Naming

- Detection rules: `DR-XXX-short-name.md` (e.g. `DR-001-suspicious-powershell.md`)
- Triage reports: `TR-XXX-short-name.md`
- Lowercase kebab-case for file names; dates as `YYYY-MM-DD`; times in UTC.

## Content Conventions

- Use Elastic Common Schema (ECS) field names in queries and docs.
- Map every detection rule to MITRE ATT&CK (tactic + technique ID).
- Every rule needs validation evidence and false-positive notes before status `Production`.
- Markdown for docs; keep exported Kibana objects as `.ndjson`.

## Git

- Small, focused commits with descriptive messages.
- Check diffs for sensitive data before committing.
