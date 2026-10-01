# Triage Report: <Alert / Incident Title>

| Field | Value |
|-------|-------|
| Ticket ID | TR-XXX |
| Analyst | <name> |
| Date / Time Detected | YYYY-MM-DD HH:MM (UTC) |
| Date / Time Closed | YYYY-MM-DD HH:MM (UTC) |
| Triggering Rule | DR-XXX – <rule name> |
| Severity | Low / Medium / High / Critical |
| Status | Open / In Progress / Escalated / Closed |
| Verdict | True Positive / Benign True Positive / False Positive |
| MITRE ATT&CK | Txxxx(.xxx) – <name> |

> Use placeholders only (`<host-01>`, `10.0.0.x`, `<user>`). No real IPs, hostnames, usernames, or credentials.

## 1. Alert Summary

One or two sentences: what fired and why.

## 2. Affected Assets

| Type | Value (sanitized) |
|------|-------------------|
| Host | <host-01> |
| User | <user> |
| Source IP | 10.0.0.x |
| Destination | |

## 3. Investigation Timeline

| Time (UTC) | Event | Source |
|------------|-------|--------|
| HH:MM | | |

## 4. Analysis

- Queries run (KQL / EQL):
- Related events / process tree:
- Enrichment (threat intel, reputation, asset context):
- Reasoning behind the verdict:

## 5. Indicators of Compromise

| Type | Value (defanged) | Notes |
|------|------------------|-------|
| | | |

## 6. Response Actions

- [ ] Containment:
- [ ] Eradication:
- [ ] Recovery:
- [ ] Escalated to:

## 7. Lessons Learned / Tuning

- Detection improvement or tuning needed:
- Runbook updates:

## 8. Evidence

Screenshots / exports (sanitized) stored alongside this report.
