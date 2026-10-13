---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd instrument observability for the MDM hub — what do you alert on (match drift, steward backlog growth, publish lag, survivorship rule error rate) versus what you dashboard?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ MDM for Batch and Streaming](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/MDM_for_Batch_and_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To instrument observability for a Master Data Management (MDM) hub, we need to ensure that the system is functioning correctly and efficiently. At an architectural level, observability involves monitoring various aspects of the MDM hub to detect issues early, allowing for proactive remediation. Key metrics include match drift, steward backlog growth, publish lag, and survivorship rule error rate. These are critical because they directly impact data quality, system performance, and user trust in the MDM hub.

First, let's define what we're alerting on versus what we dashboard:

Alert on:
Match Drift: This is a measure of how much a specific piece of master data has changed over time. High drift rates can indicate issues with source data quality or changes that haven't been reconciled properly.
Steward Backlog Growth: Over time, stewards may accumulate unaddressed tasks, leading to increased backlog growth. This alerts us to potential resource bottlenecks and ensures that critical data management activities are being handled in a timely manner.
Publish Lag: The time it takes for changes to be propagated from the source systems to the MDM hub can impact real-time business operations. Long publish lags indicate inefficiencies or issues with synchronization processes.
Dashboard:
Survivorship Rule Error Rate: This measures how often survivorship rules (rules that govern which data should remain in the system) are violated. High error rates suggest problems with rule enforcement, leading to potential data integrity issues.
Data Quality Indicators: Comprehensive metrics such as duplicate count, missing values, and invalid formats provide an overall health check of the MDM hub’s data quality.
Throughput Metrics: The rate at which data is ingested, processed, and published can help identify bottlenecks in the system.
Step-by-Step Solution:
Define Key Performance Indicators (KPIs): Identify KPIs that are critical to the performance of the MDM hub. These include match drift rates, steward backlog growth, publish lag times, and error rates related to survivorship rules.
Implement Real-Time Monitoring: Use tools like Prometheus, Grafana, or custom-built dashboards to continuously monitor these metrics in real-time. This allows for immediate detection of anomalies.
Set Thresholds for Alerts: Define thresholds that trigger alerts when KPIs exceed certain values. For example, a match drift rate above 5% over 24 hours could be an alert threshold.
Automate Alerts and Notifications: Use incident management tools like PagerDuty or OpsGenie to send notifications to relevant stakeholders (e.g., data stewards, IT support) when alerts are triggered.
Dashboard Aggregation: Create visual dashboards that aggregate these KPIs, providing a high-level overview of the MDM hub’s health. This helps in making informed decisions and identifying trends over time.

#### Closing Summary

By instrumenting observability for the MDM hub, we ensure continuous monitoring and proactive management of critical aspects such as match drift, steward backlog growth, publish lag, and survivorship rule error rate. Alerts help us detect issues early, while dashboards provide a comprehensive view of system health and performance.

#### Common Mistakes/Follow-Ups
Mistake: Focusing only on technical metrics without considering business impact.

#### Follow-Up
Ensure that the chosen KPIs have direct relevance to business outcomes.
Mistake: Ignoring historical data trends in favor of real-time alerts.

#### Follow-Up
Use time-series analysis and trend detection to provide context for alerting.

**Comparison:**

```
Metric	Threshold	Alert Condition
Match Drift Rate	> 5%	Yes
Steward Backlog Growth	> 10 items/week	Yes
Publish Lag	> 600 seconds (10 minutes)	No
Survivorship Rule Error Rate	> 2%	Yes
```

**Code:**

```python
# Example code snippet to define alert thresholds
def check_alerts(match_drift_rate, steward_backlog_growth):
    if match_drift_rate > 5:
        send_alert("High Match Drift Rate")
    if steward_backlog_growth > 10:
        send_alert("Steward Backlog Growth Exceeds Threshold")
```

**Example input / output:**

```text
Input:
- match_drift_rate = 3.2%
- steward_backlog_growth = 8 items per week
- publish_lag_seconds = 600 seconds (10 minutes)
- survivorship_rule_error_rate = 1.5%

Output:
No alerts triggered.
```

**Data:**

```json
{
  "metrics": {
    "match_drift_rate": 3.2,
    "steward_backlog_growth": 8,
    "publish_lag_seconds": 600,
    "survivorship_rule_error_rate": 1.5
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. No
6. Yes
7. No
8. Yes
9. Start
10. Is Match Drift Rate ExceedingThreshold?
11. No Alert
12. Send Alert
13. Is Steward Backlog GrowthExceeding Threshold?
14. No Alert
15. Send Alert
16. Is Publish Lag ExceedingThreshold?
17. No Alert
18. Log for Analysis but NoImmediate Action
19. Is Survivorship Rule Error RateExceeding Threshold?
20. No Alert
21. Send Alert and Log

