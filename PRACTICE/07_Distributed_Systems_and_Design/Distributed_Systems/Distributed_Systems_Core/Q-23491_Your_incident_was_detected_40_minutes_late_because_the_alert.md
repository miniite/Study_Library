---
type: "practice"
difficulty: "Deep"
---

# Your incident was detected 40 minutes late because the alert was on a symptom not the cause. Explain how you'd move to SLO-burn-rate alerting to catch gray failures early while minimizing false pages.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SLO Burn Rate Alerting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/SLO_Burn_Rate_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where an incident was detected 40 minutes late due to a symptom rather than the cause, we need to implement SLO (Service Level Objective)-based alerting with burn-rate metrics that focus on catching gray failures early while minimizing false positives. This approach involves understanding and monitoring both the overall health of the system and its failure rates at various stages.

SLO-burn-rate alerting is a strategic shift in how we monitor and manage our distributed systems, shifting from symptom-based alerts to proactive detection of issues before they turn into full-blown failures. The core idea is to establish key performance indicators (KPIs) that help us measure the rate at which services are failing or degrading over time.

#### Triaging the Question

The question asks how to move to SLO-burn-rate alerting, specifically focusing on detecting gray failures early and minimizing false pages. Gray failures occur when a system is partially degraded but not fully failed, leading to suboptimal performance or partial service disruptions. False positives are unwanted alerts that do not indicate actual problems.

#### First-Principles Solution

To implement this solution effectively:

Define SLOs: Establish Service Level Objectives for your critical services and applications.
Monitor Burn Rates: Track the rate at which these objectives are being met or violated over time.
Detect Gray Failures Early: Set thresholds to identify when services start degrading beyond acceptable levels.
Reduce False Positives: Implement heuristics and context-aware alerts that differentiate between actual failures and transient issues.
Step-by-Step Solution

Define SLOs:

For example, if your SLI (Service Level Indicator) is response time, an SLO might be to ensure 95% of requests have a response time under 200ms.

Monitor Burn Rates:

Calculate the burn rate by measuring how quickly the system is deviating from its SLOs over time.
Use rolling averages and standard deviations to understand the trend in failure rates.

Detect Gray Failures Early:

Implement alerting rules based on burn rates. For instance, if response times start increasing beyond 10% of their baseline, flag this as a potential issue.
Use anomaly detection algorithms (e.g., Z-score) to identify deviations from the norm.

Reduce False Positives:

Apply context-aware logic: consider other factors like time of day, recent deployment activity, or external conditions that might affect performance.
Implement multi-factor authentication for alerts to ensure they are genuine before acting on them.

#### Closing Summary
and Common Mistakes/Follow-Ups

By adopting SLO-burn-rate alerting, we can proactively detect issues in our distributed systems, minimizing both false positives and the time it takes to address critical incidents. The key is to balance sensitivity with specificity, ensuring that alerts are meaningful and actionable while reducing noise.

#### Common mistakes
include:

Over-relying on symptoms instead of root causes.
Setting overly strict SLOs without considering variability and context.
Failing to implement effective heuristics for false positives.

#### Follow-ups
might involve discussing specific tools (e.g., Prometheus, Grafana) used for monitoring and alerting, as well as strategies for continuous improvement in alert management practices.

**Comparison:**

```
Time (UTC)	Response Time (ms)	Baseline (ms)	Deviation (ms)	Burn Rate (%)
2023-10-01T00:00Z	150	200	-50	25
2023-10-01T01:00Z	160	200	-40	20
2023-10-01T02:00Z	170	200	-30	15
```

**Code:**

```python
# Example of calculating burn rate using rolling averages

def calculate_burn_rate(data_points, baseline):
    mean = sum(data_points) / len(data_points)
    deviation = abs(mean - baseline)
    burn_rate = (deviation / baseline) * 100
    return burn_rate
```

**Example input / output:**

```text
Input:
- Service response times over a few hours

Output:
- Burn rate calculation showing deviation from SLO
- Alert triggered if burn rate exceeds threshold
```

**Data:**

```json
{
  "service": "example-service",
  "SLI": {
    "response_time_ms": [
      {"timestamp": "2023-10-01T00:00Z", "value": 150},
      {"timestamp": "2023-10-01T01:00Z", "value": 160},
      {"timestamp": "2023-10-01T02:00Z", "value": 170}
    ],
    "SLO": 200,
    "burn_rate_threshold": 10
  }
}
```

**Flow:**

1. Yes
2. No
3. Initial Data Collection
4. Is SLI Deviation ExceedingThreshold?
5. Trigger Alert
6. Continue Monitoring
7. Notify Team + Perform RootCause Analysis

