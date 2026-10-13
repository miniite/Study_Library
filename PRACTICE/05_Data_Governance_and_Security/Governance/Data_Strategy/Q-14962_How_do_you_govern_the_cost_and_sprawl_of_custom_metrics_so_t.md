---
type: "practice"
difficulty: "Deep"
---

# How do you govern the cost and sprawl of custom metrics so teams can instrument freely but the org isn't paying for millions of never-queried time series?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Centralized vs Federated Telemetry](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Centralized_vs_Federated_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern the cost and sprawl of custom metrics in an organization while allowing teams to instrument freely, we need a balanced approach that ensures both operational flexibility and financial responsibility. The core idea is to establish clear policies, automate monitoring, and use proactive management techniques to control the proliferation of time series data.

First, let's define what the question is really asking: We are probing how to manage the growth of custom metrics in terms of cost efficiency, while ensuring teams have the freedom to instrument their applications as needed. This involves setting up robust policies, automating processes, and continuously monitoring the impact on resources.

#### Triage
Cost Management: Teams should be able to create and use custom metrics without exceeding a predefined budget.
Metric Governance: Policies need to ensure that only necessary metrics are retained and queried, minimizing unnecessary resource consumption.
Automation: Tools and scripts must automate the process of detecting unused or underutilized metrics.
User Awareness: Educate teams on best practices for metric creation and usage.

#### First-Principles Solution

Policy Framework:

Define a set of guidelines that outline when, where, and why custom metrics should be created.
Implement a tagging system to categorize metrics (e.g., by application, team, or criticality).

Automated Monitoring:

Use AIOps tools to monitor the usage patterns of time series data in real-time.
Automatically flag metrics that are infrequently queried and consider deactivating them.

Resource Management:

Implement cost-based alerting systems to notify teams when they exceed predefined thresholds.
Set up an automated process for periodically reviewing and archiving unused metrics.

User Education & Best Practices:

Organize training sessions and documentation on best practices for metric creation.
Provide tools that allow teams to easily manage their own metric sets, including deprecation guidelines.

#### Closing Summary
+ Common Mistakes
Mistake 1: Overlooking the importance of user education. Teams need clear guidance on when it’s appropriate to create new metrics.
Mistake 2: Not implementing automated monitoring and cost controls early enough in the process.
Mistake 3: Failing to establish a comprehensive policy framework that addresses both creation and maintenance of custom metrics.

By following this approach, we can ensure that teams have the flexibility they need while keeping costs under control and avoiding resource sprawl. The key is to balance operational freedom with financial responsibility through clear policies, automated tools, and continuous monitoring.

**Comparison:**

```
Policy	Description	Rules
App Perf Metrics	Metrics related to application performance	Max time series: 1000, Min query frequency: every 5 minutes
Service Health	Metrics for service health checks	Max time series: 500, Min query frequency: every 30 minutes
```

**Code:**

```python
# Example code for setting up cost-based alerting (pseudo-code)
def monitor_metric_usage(meter_id):
    # Simulate querying metric usage data from a monitoring system
    usage_data = get_metric_usage(meter_id)

    if usage_data['query_count'] < MIN_QUERY_THRESHOLD:
        send_alert(f"Metric {meter_id} has low query count: {usage_data}")
```

**Example input / output:**

```text
# Sample input and output for the cost-based alerting system

Input:
- Metric ID: "app-perf-metrics-123"
- Query count: 10 (in the past hour)

Output:
- Alert triggered: "Metric app-perf-metrics-123 has low query count: {current usage data}"
```

**Data:**

```yaml
# Example policy framework for metric creation
metrics_policy:
  - name: ApplicationPerformanceMetrics
    description: Metrics related to application performance
    rules:
      - max_time_series: 1000
      - min_query_frequency: every 5 minutes
```

**Flow:**

1. Pass
2. Fail
3. Yes
4. No
5. Team Instrumentation Request
6. Policy Check
7. Create Metric
8. Alert Team
9. Metric Usage Monitoring
10. Usage Threshold Exceeded?
11. Archive or Deprecate Metric
12. Periodic Review andMaintenance

