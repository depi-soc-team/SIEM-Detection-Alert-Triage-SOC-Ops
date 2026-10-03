# Tuning Notes

Noise sources, exclusions and pitfalls found while building views and rules. Newest entries at the bottom.

| # | Date | Source | Finding | Action |
|---|------|--------|---------|--------|
| 1 | 2026-10-02 | FortiGate (`fortinet_fortigate.log`) | Local traffic to `127.0.0.1:8000` is high-volume, low-value noise. | Excluded in views: `not destination.ip : "127.0.0.1"`. |
| 2 | 2026-10-02 | Lens tables (Windows Security) | Lens date-histogram rows with empty buckets produced misleading repeated 4732 rows. | Verify counts against raw events in Discover before triage. |
| 3 | 2026-10-02 | `elastic_agent*` datasets | These are agent self-monitoring data, not security telemetry. | Excluded from security views: `not data_stream.dataset : elastic_agent*`. |
