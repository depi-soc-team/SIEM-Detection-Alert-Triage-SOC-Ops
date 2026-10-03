# Kibana Data Views

Data views used by the SOC dashboards and saved searches. They are included in [04-dashboards/exports/soc-dashboards-v1.ndjson](../04-dashboards/exports/soc-dashboards-v1.ndjson).

| Data view | Index pattern | Used by |
|-----------|---------------|---------|
| `logs-*` | `logs-*` | SOC \| Data Health (events over time by source) |
| `SOC - Windows` | `logs-windows*,logs-system*` | SOC \| Data Health, SOC \| Windows & Identity |
| `SOC - FortiGate` | `logs-fortinet_fortigate.*` | Saved search FortiGate - Forward Traffic |

## Notes

- **Use `logs-windows*`, not `logs-windows.*`.** The pattern `logs-windows.*` did not match any data streams in the data view wizard.
- **`host.name` is stored lowercase** (e.g. `<pc-01>`), even when Fleet shows it in uppercase (`<PC-01>`). **KQL values are case-sensitive**, so `host.name : "<PC-01>"` returns nothing. Use the lowercase form.
