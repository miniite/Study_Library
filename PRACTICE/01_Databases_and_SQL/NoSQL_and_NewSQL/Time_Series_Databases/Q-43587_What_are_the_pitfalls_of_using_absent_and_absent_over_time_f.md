---
type: "practice"
difficulty: "Deep"
---

# What are the pitfalls of using absent() and absent_over_time() for dead-man's-switch alerting in a multi-tenant scraping setup?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ PromQL Pitfalls](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/PromQL_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of using absent() and absent_over_time() functions for dead-man’s switch alerting in a multi-tenant scraping setup, we need to understand their behavior and limitations within this context. This approach is often used in monitoring time-series data from different tenants (or customers) to detect when certain metrics are missing, indicating potential issues like failed scrapers or data collection problems. However, there are several pitfalls that can arise due to the nature of these functions.

Firstly, let’s briefly summarize the key concepts:

absent(): This function checks if a time series exists at all in the specified range.
absent_over_time(): This function is similar but operates on a sliding window over time and returns true for any point in time where there are no data points.

Now, let’s triage the question:

What is being probed: The interviewer is likely interested in understanding how these functions behave under different conditions, especially in a multi-tenant environment.

#### Assumptions
Each tenant has its own set of metrics.
Data collection intervals are consistent across tenants.
There might be occasional data loss due to network issues or other factors.

#### First-Principles Solution
Pitfall 1: False Positives Due to Missing Data Points

absent() and absent_over_time() functions will return true as soon as a single time series is missing within the specified range. This can lead to false positives, particularly in multi-tenant setups where data might be temporarily unavailable due to network issues or short-term outages.

For example, consider a scenario with two tenants: Tenant A and Tenant B. If Tenant B’s metrics are down for 30 seconds due to a network issue but then come back online immediately after, absent() will trigger an alert even though the data collection is expected to resume shortly.

Pitfall 2: Data Collection Delays

In multi-tenant setups, there can be delays in data collection. For instance, if Tenant C’s scraper has a slight delay and doesn’t update its metrics until after the time window considered by absent_over_time(), this could incorrectly trigger an alert.

Pitfall 3: Granularity Mismatch

The granularity of the time series (e.g., 15-second intervals) might not align with the expected data collection frequency. If a tenant’s scraper updates metrics less frequently than the database's resolution, absent_over_time() will be more prone to false positives.

Pitfall 4: Tenant-Specific Configuration Variations

Different tenants might have varying configurations for their scrapers and data collection intervals. This inconsistency can lead to unpredictable behavior when using these functions across multiple tenants.

Pitfall 5: Resource Contention and Query Performance

In a high-tenant setup, the overhead of querying absent() or absent_over_time() could become significant, especially if many tenants are being monitored simultaneously. This can strain the database resources and degrade overall performance.

#### Closing Summary

To mitigate these issues in a multi-tenant scraping setup using absent() and absent_over_time(), it is crucial to carefully consider the specific requirements and characteristics of each tenant’s data collection process. Implementing additional checks, such as custom query logic that accounts for known delays or anomalies, can help reduce false positives.

#### Common Mistakes

Overrelying on these functions without understanding their limitations.
Not considering varying data collection intervals across tenants.
Failing to implement rate limiting and performance optimizations for large-scale monitoring.

#### Follow-Up
Questions
How would you handle cases where data is temporarily missing due to network issues?
What strategies would you use to ensure consistent behavior across different tenant configurations?
Can you provide an example of a more robust approach using custom query logic?

**Comparison:**

| Tenant | Metric Name | Interval (s) |
|--------|-------------|--------------|
| A      | traffic     | 15           |
| B      | latency     | 30           |
| C      | errors      | 45           |

**Code:**

```python
# Example Custom Query Logic in Prometheus
alert "DeadManSwitch" {
  expr: absent_over_time(my_metric[5m]) > 0
  for: 1m
  labels: { severity: "critical", reason: "Data Missing" }
  annotations: { summary: "Metric my_metric is absent for more than 1 minute." }
}
```

**Example input / output:**

```text
Input (Prometheus Query):
absent_over_time(my_metric[1m])

Output:
- Tenant A: True (Data missing for 30 seconds)
- Tenant B: False (Data available within the window)
- Tenant C: True (Data missing for 45 seconds, but not in the 1-minute window)
```

**Data:**

```yaml
# Sample Data Schema
data:
  - tenant_id: A
    metric_name: traffic
    interval: 15s
  - tenant_id: B
    metric_name: latency
    interval: 30s
  - tenant_id: C
    metric_name: errors
    interval: 45s
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Collection Start
6. Tenant X Data Missing?
7. Alert Triggered
8. Continue Monitoring
9. Notify System Admin forInvestigation
10. Check if Temporary NetworkIssue
11. No Action Needed, ContinueMonitoring
12. Alert Resolved, Continue

