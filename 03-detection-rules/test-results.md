# Test Results

Expected vs. observed results for each simulated use case. Times in UTC; local lab time (UTC+3) in brackets.

| # | Use case | Date / time | Host | Commands | Expected | Observed | Result | Latency |
|---|----------|-------------|------|----------|----------|----------|--------|---------|
| 1 | New local admin | 2026-10-02 ~18:39 UTC (~21:39 UTC+3) | `<PC-01>` | `net user <test-user> /add`<br>`net localgroup administrators <test-user> /add`<br>`net user <test-user> /delete` | 4720, 4732, 4726 | 4720, 4728, 4732, 4726 | PASS | < 1 min |

## Notes

- **Test 1:** 4728 (member added to a security-enabled global group) was observed in addition to the expected events. It was logged during account creation and is consistent with the new account being added to a default group. Expected for this test; no action needed.
- Evidence: [SOC | Windows & Identity — Account & group changes](../04-dashboards/images/20261002_basel_dashboard-windows-identity-1.png).
