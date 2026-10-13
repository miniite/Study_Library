---
type: "practice"
difficulty: "Deep"
---

# Explain how you would set up an SLO with an error budget and describe what concrete decisions the team makes when the budget is nearly exhausted.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SLOs and Error Budgets](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/SLOs_and_Error_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up a Service Level Objective (SLO) with an error budget, we need to define measurable service quality targets that our application must meet. The SLO will help us understand if and when our system is failing to meet the expected performance or availability. An error budget quantifies how much downtime or latency we can tolerate before the service level agreement (SLA) is considered breached.

This question delves into how a backend team makes critical decisions as an SLO approaches its error budget limit. Let's break it down step by step:

Define the Service Level Objective (SLO): First, we need to define what our service should be achieving in terms of reliability and performance metrics such as availability, latency, or error rates.
Determine the Error Budget: The error budget is a measure of how much "error" your system can tolerate before it fails to meet its SLO. For example, if you have an SLA requiring 99% uptime, your error budget might be 1% downtime.

As the team approaches their error budget limit, several concrete decisions and actions will need to be taken:

Performance Monitoring: Continuously monitor performance metrics such as response time, request latency, and error rates. Tools like Prometheus, Grafana, or Datadog can help visualize these metrics in real-time.
Root Cause Analysis (RCA): When the system is approaching its budget limit, it's crucial to perform an RCA to identify the root cause of any issues. This might involve analyzing logs, tracing requests with tools like Jaeger, and understanding which parts of the application are causing the most problems.
Capacity Planning: Review your infrastructure capacity, including CPU, memory, network bandwidth, and storage. If necessary, scale up resources or migrate to more efficient architectures (e.g., microservices). Use tools like Kubernetes for dynamic scaling.
Optimize Application Code: Look for bottlenecks in the application code that might be causing increased latency or error rates. This could involve refactoring critical paths, optimizing database queries, or implementing caching strategies.
User Experience Mitigation: If errors are becoming frequent, consider mitigating their impact on users. For example, implement retries, circuit breakers, or fallback mechanisms to ensure a better user experience even if the system is under stress.

When the error budget is nearly exhausted, here’s what happens:

Notify Stakeholders: Immediately inform the relevant stakeholders about the approaching budget limit and potential risks.
Implement Defensive Measures: Deploy defensive measures such as rate limiting, traffic shifting, or fallback strategies to protect against further degradation of service.
Plan for Downtime or Degraded Service: Prepare for temporary downtime or degraded service by having a plan in place. This might involve performing maintenance during off-peak hours or even temporarily shutting down non-critical services.
Investigate and Mitigate Issues: Prioritize addressing the root causes of the issues to prevent further degradation. Use A/B testing if necessary to validate changes without causing widespread disruption.

#### Closing Summary
Setting up an SLO with an error budget involves defining clear service objectives, continuously monitoring performance, performing root cause analysis, optimizing infrastructure and application code, and taking proactive measures when the budget is nearly exhausted. It’s a strategic process that requires constant vigilance and adaptation to ensure high-quality service delivery.

#### Common Mistakes/Follow-ups

Overlooking continuous monitoring: Relying too heavily on historical data without real-time visibility can lead to surprises.
Ignoring capacity planning: Not having adequate resources when the system is under stress can quickly consume your error budget.
Failing to implement defensive measures: Without strategies like retries and circuit breakers, small issues can compound into larger problems.
Poor communication: Stakeholders must be kept informed to manage expectations and avoid panic.

**Comparison:**

```
SLO Metric	Target	Current Value
Response Time	50ms	65ms
Error Rate	0.01%	0.12%
```

**Code:**

```python
# Example of setting up an SLO with an error budget using Prometheus metrics
from prometheus_client import Counter

error_counter = Counter('service_errors', 'Number of errors in the service')
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Metrics data (e.g., response time, error rate) | Alert: Error budget nearly exhausted |
| Root cause analysis report | Action plan for mitigation and optimization |

**Data:**

```json
{
  "slo": {
    "name": "99.99% Uptime",
    "target": "0.01%", // 1% error budget
    "metrics": [
      { "metric_name": "response_time", "target": "50ms" },
      { "metric_name": "error_rate", "target": "0.01%" }
    ]
  }
}
```

**Flow:**

1. Monitor SLO Metrics
2. Analyze Root Causes
3. Optimize Application Code
4. Scale Infrastructure
5. Implement DefensiveMeasures
6. Notify Stakeholders
7. Plan for Downtime

