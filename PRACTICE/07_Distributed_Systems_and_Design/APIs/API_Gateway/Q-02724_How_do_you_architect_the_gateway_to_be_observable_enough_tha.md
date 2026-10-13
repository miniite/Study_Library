---
type: "practice"
difficulty: "Deep"
---

# How do you architect the gateway to be observable enough that you can attribute a customer-reported 'the app is slow' complaint to a specific route, region, upstream, and time window within minutes, across a fleet serving billions of requests a day, without drowning in cardinality?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Cost Metering and Budgets](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/Gateway_Cost_Metering_and_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an observability solution for an API Gateway that can attribute customer-reported issues to specific routes, regions, upstream services, and time windows within minutes, we need to ensure our system is highly scalable, yet detailed enough to provide the necessary insights. The key challenge here lies in balancing fine-grained metrics with cardinality concerns, ensuring we can quickly identify problematic areas without overwhelming ourselves with too much data.

We are probing for an understanding of how to implement a robust observability framework that leverages distributed tracing and metric aggregation techniques. We will need to consider the following assumptions:

The API Gateway is serving billions of requests per day.
We need to support multiple regions, potentially thousands of microservices, and handle customer complaints in real-time.
We must ensure data retention for at least a few hours or days to analyze trends.

#### First Principles Solution

Metrics Collection:

We need to instrument every API endpoint with detailed metrics that capture request latency, error rates, and throughput.
For each request, we should collect metadata such as user ID (if available), IP address, timestamp, region, and route details.

Distributed Tracing:

Implement a distributed tracing system like Jaeger or OpenTracing to track the entire journey of a request through multiple services.
This will help us understand which microservices are contributing to increased latency.

Metric Aggregation:

Use time-series databases (e.g., InfluxDB, TimescaleDB) to store and aggregate metrics at various levels (e.g., per service, per region).
Implement a retention policy that ensures data is stored for an appropriate period while minimizing storage costs.

Anomaly Detection:

Deploy anomaly detection mechanisms using statistical models or machine learning techniques.
Set up alerts based on predefined thresholds to notify the engineering team about potential issues.

User-Centric Views:

Develop dashboards that allow users (e.g., support engineers) to query metrics and trace data in a user-friendly way.
Ensure these dashboards can filter by region, time window, route, and other relevant dimensions.
Step-by-Step Solution

Instrument Metrics at the Gateway Level:

metrics
API Gateway
Ingress Route
Upstream Service
Outbound Route
Metric Storage
At each step, we collect relevant metrics and send them to a central metric storage system.

Implement Distributed Tracing:

traces
API Gateway
Tracer Service
Collector Service
Distributed Trace Store
Use the tracer service to inject trace IDs and span contexts into every request.
Collect these traces in a central store where they can be correlated across services.

Set Up Alerting and Notifications:

notifications
API Gateway
Metric Aggregation Service
Alerting System
Engineering Team
Define rules for alerting based on metrics like latency or error rates.
Integrate with an alerting system to notify the engineering team in real-time.

#### Closing Summary

In summary, we need a combination of detailed metrics collection, distributed tracing, and anomaly detection to provide observability at scale. By carefully designing our systems, we can ensure that customer complaints can be quickly attributed to specific routes, regions, and upstream services without overwhelming ourselves with too much data.

#### Common Mistakes
and Follow-Ups:

Mistake: Focusing only on metrics without implementing distributed tracing can lead to blind spots in understanding the request flow.

#### Follow-Up
How would you handle edge cases where multiple microservices are involved, and a single slow service could be part of an otherwise fast pipeline?

**Comparison:**

| Route | Region | Latency (ms) | Error Rate (%) |
|-------|--------|-------------|---------------|
| /v1/user/profile | us-west-2 | 500 | 0.01 |
| /v1/payment/transaction | eu-central-1 | 300 | 0.001 |

**Code:**

```python
# Example Python code for collecting metrics
from opentracing.ext import tags
import jaeger_client

tracer = jaeger_client.JaegerTracer(
    service_name='api-gateway',
    reporter=jaeger_client.Reporter()
)

def handle_request(request):
    with tracer.start_span('handle-request') as span:
        span.set_tag(tags.SPAN_KIND, tags.SPAN_KIND_SERVER)
        # Collect metadata and metrics
        ...
```

**Example input / output:**

```text
Input:
API Gateway receives a request to /v1/user/profile

Output:
{
  "metrics": {
    "latency_ms": 500,
    "error_rate": 0.01,
    "throughput_per_second": 100
  },
  "metadata": {
    "user_id": "12345",
    "ip_address": "192.168.1.1",
    "timestamp": "2023-10-01T14:30:00Z",
    "region": "us-west-2",
    "route": "/v1/user/profile"
  },
  "trace_id": "abcd1234efgh5678",
  "span_id": "ijkl90abcd12ef"
}
```

**Data:**

```json
{
  "metrics": {
    "latency_ms": 500,
    "error_rate": 0.01,
    "throughput_per_second": 100
  },
  "metadata": {
    "user_id": "12345",
    "ip_address": "192.168.1.1",
    "timestamp": "2023-10-01T14:30:00Z",
    "region": "us-west-2",
    "route": "/v1/user/profile"
  }
}
```

**Flow:**

1. metrics and traces
2. aggregated data
3. trace IDs, spans
4. User Request
5. API Gateway
6. Metric Storage + Tracer Service
7. Distributed Trace Store
8. Alerting System

