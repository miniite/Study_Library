---
type: "practice"
difficulty: "Intermediate"
---

# Your on-call got paged because averaged latency dashboards looked fine but customers reported slowness. Explain why aggregate metrics hide per-tenant tail latency and how you'd instrument to surface it.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Aggregation Hiding Tail Problems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Aggregation_Hiding_Tail_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed system, aggregate metrics like average latency can provide an overall health check, but they often mask severe issues affecting specific users or tenants. For example, if 95% of requests are fast and only 5% experience high latency, the average might look good even though critical customers are experiencing significant delays. To surface these tail latencies, we need to implement tenant-specific instrumentation that captures detailed performance metrics at a per-tenant level.

To explain further, let’s break down the problem and solution step-by-step:

Understanding the Problem: Aggregated metrics like average latency can be misleading when they don't reflect the real experience of all users or tenants. We need to identify why certain tenants are experiencing poor performance while others aren’t.

#### Assumptions
and Context:

The system is distributed with multiple tenants.
Performance issues are intermittent, making them difficult to detect through standard monitoring.
The current monitoring setup only aggregates metrics across all users or services without distinguishing between different tenants.

#### First Principles Solution

Implement tenant-specific dashboards: Use the existing infrastructure and add new metrics that break down latency per tenant.
Use distributed tracing: Integrate a distributed tracing system like Jaeger or Jaegar Lite to capture end-to-end requests, identifying which requests are slow for specific tenants.
Enhance logging: Add detailed logs at key points in the service flow to trace issues and identify bottlenecks affecting specific tenants.

Detailed Steps:

Add Tenant IDs: Ensure each request includes a tenant ID so that metrics can be collected and displayed per tenant.
Metrics Collection: Collect detailed latency metrics for each tenant using Prometheus or similar monitoring tools. For instance, use histograms to capture the distribution of latencies.
Distributed Tracing Setup: Deploy a tracing system that captures the full request path in distributed services. This will help identify where the latency spikes occur and which tenants are affected.
Log Aggregation: Set up log aggregation for detailed inspection. Use tools like ELK Stack or Graylog to collect, index, and query logs efficiently.

#### Implementation
Example: Suppose we have a service that processes requests from multiple tenants. We can add the following instrumentation:

PYTHON
# Python example using OpenTracing API
from opentracing.ext import tags

def handle_request(request):
    # Start a span for this request
    with tracer.start_active_span("handle_request", child_of=request_context) as scope:
        # Add tenant ID to the span tags
        scope.span.set_tag('tenant_id', request.tenant.id)

        # Perform request processing
        result = process_request(request)

    return result

# Example histogram setup in Prometheus
histogram.observe(request.latency_ms, labels={'tenant': request.tenant.id})

Monitoring and Alerts:

Set up alerts based on the new metrics to notify the team when tail latencies exceed a certain threshold.
Use Grafana dashboards to visualize these metrics, making it easier to spot issues.

#### Common Mistakes
and Follow-Ups:

Failing to distinguish between tenant-specific metrics can lead to blind spots in monitoring.
Not having a distributed tracing system means you won't be able to trace the full request path, potentially missing critical performance bottlenecks.
Overlooking log aggregation can make it difficult to diagnose issues after they occur.

**Comparison:**

```
Metric	Description
Average Latency	Overall average response time across all tenants.
95th Percentile Latency	95% of requests are faster than this value; useful for understanding overall performance but hides severe issues.
Tenant-Specific Histograms	Detailed latency distributions per tenant, identifying performance issues affecting specific users or services.
```

**Code:**

```python
# Python example using OpenTracing API
from opentracing.ext import tags

def handle_request(request):
    with tracer.start_active_span("handle_request", child_of=request_context) as scope:
        scope.span.set_tag('tenant_id', request.tenant.id)

        result = process_request(request)

    return result

# Example histogram setup in Prometheus
histogram.observe(request.latency_ms, labels={'tenant': request.tenant.id})
```

**Example input / output:**

```text
Input:
- Request with tenant ID = TENANT-123

Output:
- Metrics collected for TENANT-123, including histogram data and tracing information.
- Alerts triggered if tail latencies exceed thresholds.
```

**Data:**

```json
{
  "metrics": {
    "average_latency": 10ms,
    "95th_percentile_latency": 500ms,
    "tenant_id": "TENANT-123"
  }
}
```

**Flow:**

1. Yes
2. No
3. Request
4. Tenant ID?
5. Process Request and Log
6. Tenant-Specific MetricsCollection
7. Distributed Tracing SpanCreation
8. Log Details of Request Path
9. Distribute Traced RequestsAcross Services
10. Metric Aggregation for AllTenants

