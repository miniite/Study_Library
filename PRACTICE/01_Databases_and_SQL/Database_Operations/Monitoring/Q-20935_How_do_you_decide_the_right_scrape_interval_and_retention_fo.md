---
type: "practice"
difficulty: "Deep"
---

# How do you decide the right scrape interval and retention for database metrics without exploding storage?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Percentiles vs Averages](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Percentiles_vs_Averages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine the optimal scrape interval and retention strategy for database metrics, we need to balance the trade-offs between storage efficiency, data freshness, and query performance. A key aspect is understanding how different intervals and retention policies impact these factors. We'll start with a brief overview of the process, then dive into first-principles reasoning.

TL;DR

The right scrape interval should be balanced between capturing recent changes and avoiding excessive resource consumption. Retention periods need to strike a balance between storing enough data for thorough analysis while minimizing storage costs. A common approach is to use a tiered architecture where more critical metrics are retained longer, and less critical ones are discarded faster.

#### Triaging the Question

The question is asking how to set scrape intervals (how often we collect new metric data) and retention periods (how long we keep collected data). The interviewer is probing your understanding of trade-offs in database monitoring systems. You'll need to consider factors like storage costs, query performance, and the importance of having recent versus historical data.

#### First-Principles Solution
1. Define Key Metrics and Their Urgency

First, identify which metrics are critical for immediate alerting and troubleshooting. For example, transaction counts or error rates might be more urgent than slow query times. Less critical metrics can be monitored less frequently.

2. Understand Storage Costs

Storage costs vary based on the size of the database and the frequency of writes. A shorter scrape interval means more frequent writes, which can increase storage costs significantly. For instance, if you collect metrics every second, they will take up a lot more space compared to collecting them every five minutes.

3. Evaluate Query Performance Impact

Frequent scrapes mean that your database will be queried more often for recent data. This can affect performance, especially with large datasets. Conversely, too infrequent scraping might make it difficult to spot trends and issues quickly.

4. Tiered Retention Strategy

Implement a tiered retention strategy where critical metrics are retained longer (e.g., days or weeks), while less critical ones are discarded after a shorter period (e.g., hours). This helps balance the need for historical data with storage constraints.

#### Example Scenario
s
Critical Metrics: These could include application performance indicators like response times, error rates, and transaction counts. They should be monitored frequently (every few minutes) and retained for at least 30 days.
Less Critical Metrics: Examples might include daily usage statistics or slow query logs. These can be collected less often (e.g., every hour) and retained for a week.

#### Closing Summary

In summary, the optimal scrape interval and retention strategy depend on the importance of the metrics being monitored. A balanced approach involves using frequent scraping for critical data and less frequent scraping with shorter retention periods for non-critical data. This ensures that both recent and historical insights are available without overwhelming storage resources.

#### Common Mistakes/Follow-Ups
Ignoring Critical Metrics: Failing to prioritize critical metrics can lead to missing important issues.
Overly Aggressive Scraping: Too frequent scraping can unnecessarily increase storage costs and degrade query performance.
Uniform Retention Policies: Treating all data uniformly without considering its importance is inefficient.
Triaging Follow-Ups

If asked about specific tools or strategies, mention using monitoring dashboards like Grafana to visualize trends. Discuss how you might implement alerting thresholds based on historical data patterns. If asked about scaling considerations, explain the need for scalable storage solutions and possibly archiving older data to a lower-cost storage tier.

**Comparison:**

```
Metric Type	Scrape Interval (min)	Retention Period (days)
response_time	5	30
error_rate	10	45
slow_queries	60	24
```

**Code:**

```sql
-- Example SQL Query to collect metrics every 5 minutes
CREATE SCHEDULED TASK metric_collection AS
INSERT INTO database_metrics (metric_type, value)
VALUES ('response_time', <current_value>)
ON EVERY 5 MINUTES;
```

**Example input / output:**

```text
Input: Metric type = response_time, Time Interval = 5 minutes, Retention Period = 1 week
Output: Data is collected every 5 minutes and stored for 7 days.
```

**Data:**

```json
{
  "metrics": {
    "critical": ["response_time", "error_rate"],
    "non_critical": ["slow_queries", "daily_usage"]
  },
  "retention_periods": {
    "critical": {"days": 30},
    "non_critical": {"hours": 24}
  }
}
```

**Flow:**

1. Define Critical Metrics
2. Monitor Freely
3. Identify Less Critical Metrics
4. Collect Less Frequently
5. Retain for Longer Periods(Days)
6. Discard After Shorter Periods(Hours)

