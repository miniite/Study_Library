---
type: "practice"
difficulty: "Deep"
---

# Your auto-scaling remediation reacts to a CPU alert, but the workload is actually latency-bound behind a slow dependency, so scaling out did nothing and made recovery slower. Explain why the scaling signal is wrong and what metric you'd trigger remediation on instead.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Metastable Failures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/Metastable_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, auto-scaling policies are crucial to ensure optimal resource utilization while maintaining service performance. However, they can be tricky when the underlying issue isn't directly related to CPU usage but rather latency in a slow dependency. In this scenario, scaling out based on CPU metrics is ineffective because it doesn’t address the root cause of the problem. To fix this, we need to focus on a different metric that better reflects the actual performance bottleneck.

Triaging the Issue

The core issue here is that auto-scaling policies are designed to react to changes in resource usage (like CPU), but they often fail when the limiting factor is external and not directly related to internal resource consumption. Specifically, if the system is latency-bound due to a slow dependency, simply scaling out more instances won’t help because the new instances will also face the same bottleneck.

Why the Scaling Signal Is Wrong
CPU Metrics Don't Reflect Latency: CPU usage might be low because the application isn't doing intensive computations; it's waiting for responses from another service or database.
Scale-Up Isn't Sufficient: Adding more resources doesn’t alleviate latency issues, which are often caused by network bottlenecks, slow external services, or I/O operations.
What Metric to Trigger Remediation On Instead

To effectively address this issue, we need to monitor and act on the actual source of the performance degradation. The key metric here is latency—specifically, end-to-end request latency from the application to its dependencies.

Step-by-Step Solution
Identify Latency Sources: Use distributed tracing tools like Jaeger or Zipkin to identify which services are contributing most to the overall latency.
Monitor API Response Time: Implement metrics to track response times of critical APIs and external services.
Set Up Alerting on Latency: Configure alerting mechanisms to notify when latency exceeds a certain threshold, indicating that the system is becoming bottlenecked by its dependencies.

#### Example
Alert Rule
YAML
# Example Prometheus rule for monitoring API latency
groups:
  - name: api-latency-monitoring
    rules:
      - alert: APIResponseTimeHigh
        expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[1m])) by (le))
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "API response time is high"
          description: "The 95th percentile API response time exceeds the expected threshold."

#### Closing Summary

By focusing on end-to-end latency rather than CPU usage, we can better understand and address the true performance bottlenecks in distributed systems. This approach ensures that scaling actions are more likely to be effective by aligning with the actual performance metrics.

#### Common Mistakes
and Follow-Ups
Ignoring External Dependencies: Many engineers focus solely on internal metrics without considering external factors.
Misinterpreting CPU Metrics: Assuming high CPU usage always indicates a need for more resources, when it might just mean waiting for responses from other services.
Failing to Use Distributed Tracing Tools: Without proper visibility into the entire system's performance, pinpointing the exact bottleneck can be challenging.

In summary, effective monitoring and remediation in distributed systems require a holistic approach that includes tracking end-to-end latency and understanding external dependencies.

**Comparison:**

```
Trace ID	Span ID	Operation Name	Duration (ms)
1234567890abcdef	abcdef123456	"/api/v1/resource"	50
1234567890abcdef	bcdef123456	"backend-service request"	9500
```

**Code:**

```yaml
# Example Prometheus alert rule for API response time
groups:
  - name: api-latency-monitoring
    rules:
      - alert: APIResponseTimeHigh
        expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[1m])) by (le))
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "API response time is high"
          description: "The 95th percentile API response time exceeds the expected threshold."
```

**Example input / output:**

```text
Input: A user request is made to a web service.
Output: The request takes 10 seconds to complete, with the majority of time spent waiting for an external backend service.
```

**Data:**

```yaml
# Example distributed tracing configuration using Jaeger
services:
  - name: web-service
    tags:
      service_name: web-service

  - name: backend-service
    tags:
      service_name: backend-service

spans:
  - trace_id: "1234567890abcdef"
    parent_id: null
    span_id: "7890abcdef123456"
    operation_name: "/api/v1/resource"
    references:
      - ref_type: CHILD_OF
        trace_id: 1234567890abcdef
        span_id: abcdef1234567890
    start_time: "2023-10-01T10:00:00Z"
    duration: 50ms
    tags:
      http.status_code: 200
      http.method: GET
```

**Flow:**

1. User Request
2. B/WebAPI
3. B
4. Backend Service
5. Database
6. External API
7. Latency bottleneck
8. Service Times Out
9. System Responds Slowly

