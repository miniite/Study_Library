---
type: "practice"
difficulty: "Deep"
---

# How would you implement observability SLOs and alerting on the gateway (availability, latency, error budget) so on-call gets actionable pages and not noise?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Cost Metering and Budgets](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/Gateway_Cost_Metering_and_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement observability Service Level Objectives (SLOs) and alerting on an API Gateway, we need to ensure that our gateway is robust, reliable, and provides clear, actionable insights when issues arise. This involves setting up comprehensive monitoring for availability, latency, and error rates, defining SLOs based on these metrics, and configuring alerts to notify the on-call team appropriately.

#### Triaging the Question

The question asks us to implement observability SLOs and alerting mechanisms on an API Gateway with a focus on reducing noise while ensuring that the on-call team receives actionable pages. The key aspects include:

Availability: Ensuring high uptime for the gateway.
Latency: Keeping response times within acceptable bounds.
Error Budget: Managing error rates without exceeding predefined thresholds.

#### Assumptions
We are using a microservices architecture with an API Gateway in front of various services.
The gateway is built on a modern framework such as Kubernetes or AWS App Mesh, which supports advanced observability features.
We have a clear understanding of the application's critical paths and expected performance characteristics.

#### First-Principles Solution

Define SLOs:

Availability: Set an SLO for 99.5% uptime over any 4-week period, meaning no more than 2 hours of downtime in those 4 weeks.
Latency: Ensure that the average response time does not exceed 100ms under normal conditions and does not exceed 200ms during peak times.
Error Budget: Allow for a maximum error rate of 1% per month, which translates to an annual budget of about 8 hours.

Choose Monitoring Tools:

Use Prometheus for time-series metrics collection and Grafana for visualization.
Integrate with Alertmanager for alerting based on SLO thresholds.
Employ Jaeger or Zipkin for tracing requests and identifying bottlenecks.

Implement Metrics Collection:

Set up custom metrics in the gateway to capture relevant data such as request counts, response times, error rates, etc.
Use OpenTelemetry SDKs to instrument the gateway and backend services to ensure comprehensive coverage.

Alerting Mechanisms:

Configure alerts for availability (e.g., uptime checks).
Set up alerts for latency issues that exceed predefined thresholds.
Define alert rules for error rates that approach or exceed the error budget, ensuring that critical errors are flagged early.

Actionable Alerts and Notifications:

Design a system where alerts are sent via pager duty (or similar on-call management tools) with clear severity levels.
Ensure that low-severity notifications do not flood the team; instead, prioritize critical issues that could affect user experience or service availability.

Testing and Validation:

Regularly run load tests to validate SLOs under different conditions.
Monitor historical data to fine-tune thresholds and ensure they are realistic yet strict enough to catch issues early.

#### Closing Summary

By implementing a robust observability framework, we can ensure that the API Gateway performs reliably and provide actionable insights when issues arise. This involves setting clear SLOs, leveraging modern monitoring tools, and configuring alerts appropriately to avoid noise while ensuring critical incidents get immediate attention from the on-call team.

#### Common Mistakes
and Follow-Ups
Mistake: Setting unrealistic thresholds that lead to constant false positives or negatives.

#### Follow-Up
Regularly review and adjust SLOs based on actual performance data.
Mistake: Failing to differentiate between different types of alerts (e.g., informational vs. critical).

#### Follow-Up
Implement a triage system within the alerting process to ensure that only critical issues are escalated.

**Comparison:**

```
Metric	Thresholds
Availability	>98.5% (4 weeks)
Latency	<=100ms (normal), <=200ms (peak)
Error Rate	<1% per month
```

**Code:**

```bash
# Example Alertmanager configuration for error budget
groups:
- name: "api-gateway"
  rules:
  - alert: HighErrorRate
    expr: sum by (job) (rate(http_requests_total[5m])) * on(job) http_response_code >= 400 without (error_code)
    for: 1h
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      description: "Error rate is exceeding the allowed budget. Investigate immediately."
```

**Example input / output:**

```text
Input (Sample Data):
- Total requests: 60,000
- Successful requests: 58,900
- Error requests: 1,100

Output:
- Availability SLO: Meets target with 98.3% uptime.
- Latency SLO: Passes with an average response time of 90ms.
- Error Rate SLO: Exceeds target by 5%, triggering alerts for further investigation.

Dry Run (Example Alert):
- HighErrorRate alert triggered due to a sudden increase in error rate, indicating potential issues that need immediate attention.
```

**Data:**

```json
{
  "gateway": {
    "name": "api-gateway",
    "metrics": [
      { "type": "request_count", "value": 5000 },
      { "type": "response_time", "value": 120ms }
    ],
    "slos": {
      "availability": 99.5%,
      "latency": "<=100ms",
      "error_rate": "<=1%"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Request
6. API Gateway
7. Backend Service
8. Response to User
9. Is Request Successful?
10. Log Success, IncrementMetrics
11. Error Rate Exceeds SLO?
12. Alert On-Call Team
13. Monitor and Log Error

