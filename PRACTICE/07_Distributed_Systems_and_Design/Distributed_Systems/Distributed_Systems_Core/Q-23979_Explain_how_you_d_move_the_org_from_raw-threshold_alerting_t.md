---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd move the org from raw-threshold alerting to multi-window SLO-burn-rate alerting as a distributed service. Why do raw thresholds cause both false pages and missed slow-burn gray failures, and how do you set the windows so on-call is paged early without alert fatigue?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ SLO Burn Rate Alerting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/SLO_Burn_Rate_Alerting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In transitioning from raw-threshold alerting to multi-window SLO-burn-rate alerting as a distributed service, we're moving beyond simple threshold-based alerts that can lead to both false positives and missed detections of slow burn issues. The goal is to introduce a more sophisticated approach that continuously monitors service-level objectives (SLOs) across multiple time windows to ensure reliable on-call handling without overwhelming the team with unnecessary alerts.

#### Triaging the Question

The core of this question involves understanding the limitations of raw-threshold alerting and how multi-window SLO-burn-rate monitoring can address these issues. The interviewer is likely probing your knowledge of distributed systems, service-level objectives (SLOs), and best practices in alert management. We need to make assumptions about the existing system's architecture and data availability.

#### First-Principles Solution

Understanding Raw Threshold Alerting: Raw-threshold alerts are based on simple conditions like CPU usage > 80% for more than X minutes. While straightforward, they can lead to two main issues:

False Positives: Spikes or temporary bursts of load might trigger alerts that do not reflect the true state of the system.
Missed Slow Burn Issues: Persistent but gradual degradation in performance may go unnoticed until it's too late.

Introducing Multi-Window SLO-Burn-Rate Alerting:

SLOs and Burn Rate: Define clear SLOs (e.g., 95th percentile response time ≤ 100ms) and calculate burn rates to track how close the system is to these targets.
Multi-Time Windows: Use multiple time windows (short-term, medium-term, long-term) to capture different aspects of performance:
Short-term: Capture recent, sudden changes that might indicate a temporary issue.
Medium-term: Reflect recent trends and help in diagnosing ongoing issues.
Long-term: Provide historical context and help identify gradual drifts.

Distributed Service Architecture:

Microservices: Ensure the alerting service is decoupled from the core services to minimize impact.
Stateless Design: Make the alerting service stateless to handle failures gracefully and ensure high availability.
Rate Limiting and Aggregation: Implement rate limiting to prevent alert fatigue and aggregate data across multiple sources.

#### Solution
Steps

Define SLOs:

MARKDOWN
SLO: 95th percentile response time ≤ 100ms for all requests.

Calculate Burn Rates:

Short-term: Use a sliding window of the last hour to detect recent issues.
Medium-term: Use a rolling window of the last 24 hours to capture trends.
Long-term: Use historical data over the past month to identify gradual changes.

Implement Multi-Window Monitoring:

MARKDOWN
Short-Term Window (1h):
- Calculate current burn rate against SLO.
- If burn rate > 90%, trigger immediate alert for short-term issues.

Medium-Term Window (24h):
- Monitor trend over the last day.
- Alert if there is a significant deviation from historical performance.

Long-Term Window (30d):
- Analyze long-term trends to identify gradual degradation.
- Trigger alerts for sustained deviations from baseline performance.

Alerting Service Architecture:

Data Collection
Metric Aggregation
Distributed Alerting Service
On-Call Notification

Decoupling and Scalability:

Use a message queue to decouple data collection from alert processing.
Implement retries, backoff strategies, and circuit breakers to ensure the service remains robust.

#### Closing Summary

By moving from raw-threshold alerting to multi-window SLO-burn-rate monitoring as a distributed service, we address both false positives and missed detections. This approach provides a more nuanced view of system performance, allowing for early detection of issues while reducing alert fatigue. The key is to define clear SLOs, implement multiple time windows, and ensure the service architecture supports high availability and scalability.

#### Common Mistakes/Follow-Ups
False Positives: Overly aggressive short-term alerts can lead to false positives.
Alert Fatigue: Insufficient rate limiting can result in overwhelming on-call teams with too many alerts.
Complexity Management: Managing multiple windows requires careful tuning of thresholds and time intervals.

Would you like me to elaborate on any specific aspect or consider an example scenario?

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Example Gantt Chart for SLO Monitoring Service
    section Data Collection
        2023-10-01T14:00 -> 2023-10-01T15:00 Collect data points
    section Metric Aggregation
        after Collect data points
        2023-10-01T16:00 -> 2023-10-01T17:00 Calculate burn rates and triggers
    section Alerting Service
        after Calculate burn rates and triggers
        2023-10-01T18:00 -> 2023-10-01T19:00 Process alerts and notify on-call team
```

**Code:**

```python
# Example code for SLO calculation (simplified)
def calculate_slo_burn_rate(data_points, window_size):
    # Calculate 95th percentile response time
    p95 = np.percentile(data_points, 95)
    current_time = datetime.now()

    # Short-term burn rate check
    if p95 > 100:
        return True, "Short-term alert: P95 > 100ms"

    # Medium-term trend check
    recent_data = data_points[max(0, len(data_points) - window_size):]
    trend_change = np.mean(recent_data) - np.mean(data_points)
    if abs(trend_change) > 2:
        return True, "Medium-term alert: P95 trend change"

    # Long-term historical check
    long_term_avg = np.mean(data_points[-30:])
    if p95 > (1.2 * long_term_avg):
        return True, "Long-term alert: SLO drift detected"

    return False, None
```

**Example input / output:**

```text
| Input Data Points | Short-term SLO Burn Rate Alert? | Medium-term Trend Alert? | Long-term Historical Alert? |
|-------------------|--------------------------------|-------------------------|----------------------------|
| [98, 103, 104, ...] | Yes (P95 > 100ms)               | No                      | No                         |

Input: [98, 103, 104, ..., 120]
Output:
- Short-term Alert: P95 > 100ms
- Medium-term: No alert
- Long-term: No alert

Input: [102, 107, 106, ..., 108]
Output:
- Short-term: No alert
- Medium-term Alert: P95 trend change detected
- Long-term: No alert
```

**Data:**

```yaml
- data_points:
  - 98
  - 103
  - 104
  - 105
  - 106
  - 107
  - 102
  - 99
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Collect Data Points
8. Metric Aggregation
9. SLO Calculation
10. Short-term Alert Trigger?
11. Notify On-Call Team
12. Metric Aggregation
13. Check Medium-term Trend
14. Notify On-Call Team
15. Metric Aggregation
16. Check Long-term Historical
17. Notify On-Call Team
18. Rate Limiting and Backoff

