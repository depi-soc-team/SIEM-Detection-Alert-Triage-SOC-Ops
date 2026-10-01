# Rule: <Rule Name>

| Field | Value |
|-------|-------|
| Rule ID | DR-XXX |
| Author | <name> |
| Created / Updated | YYYY-MM-DD / YYYY-MM-DD |
| Status | Draft / Testing / Production / Deprecated |
| Severity | Low / Medium / High / Critical |
| Risk Score | 0–100 |
| Rule Type | Query (KQL/EQL/ES\|QL) / Threshold / EQL Sequence / ML / Indicator Match |
| MITRE ATT&CK | Tactic: TAxxxx – <name> · Technique: Txxxx(.xxx) – <name> |

## Description

What behavior this rule detects and why it matters.

## Data Sources

- Integration / index pattern: `logs-<integration>.*`
- Required fields (ECS): `event.code`, `process.name`, ...

## Query

```
<query here>
```

- Schedule: runs every `5m`, look-back `+1m`
- Threshold / sequence settings (if any):

## Test / Validation

| Step | Detail |
|------|--------|
| Simulation method | e.g. Atomic Red Team test ID, manual command |
| Expected result | Alert fires with ... |
| Actual result | Pass / Fail — notes |
| Evidence | Screenshot or link (sanitized) |

## False Positives & Tuning

- Known benign triggers:
- Exclusions applied:

## Response Guidance

Initial triage steps for an analyst (link to runbook in `05-triage/` if available).

## References

- 
