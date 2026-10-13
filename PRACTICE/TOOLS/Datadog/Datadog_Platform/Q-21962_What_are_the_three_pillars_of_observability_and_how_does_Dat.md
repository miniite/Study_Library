---
type: "practice"
difficulty: "Basic"
---

# What are the three pillars of observability and how does Datadog map to each of them?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Database Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Database_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The three pillars of observability are Tracing, Logging, and Metrics. These fundamental concepts ensure that your systems can be observed, understood, and improved over time. Datadog is a powerful monitoring platform that excels in all these areas.

#### Triaging the Question

The question asks you to explain the three pillars of observability and how they relate specifically to Datadog. This means understanding what each pillar entails and then identifying where Datadog provides solutions for them.

Key Concepts and Assumptions
Tracing: This involves tracking requests as they move through a system, providing visibility into service-to-service communication.
Logging: This includes collecting application logs to understand the state of systems at different points in time.
Metrics: These are numerical values that provide insight into the performance and health of your applications.
Step-by-Step Solution

Tracing:

Definition: Tracing involves following a request through its journey across multiple services, allowing you to understand dependencies and identify bottlenecks.
Datadog’s Approach: Datadog supports distributed tracing via open standards like OpenTelemetry (OTel) or OpenTracing. It collects trace data from applications and correlates it to provide end-to-end visibility.

Logging:

Definition: Logging involves collecting application logs, which can be analyzed for debugging, monitoring, and alerting.
Datadog’s Approach: Datadog offers robust log management capabilities with real-time search and filtering tools. It supports ingestion from various sources and integrates well with other observability data.

Metrics:

Definition: Metrics are numerical values that provide a quantitative view of system performance, such as response times or error rates.
Datadog’s Approach: Datadog collects and aggregates metrics from applications, infrastructure, and databases. It offers customizable dashboards for visualizing these metrics and setting up alerts based on threshold conditions.

#### Closing Summary

In summary, Datadog maps to the three pillars of observability by providing comprehensive solutions in each area:

Tracing through distributed tracing capabilities.
Logging with powerful log management tools.
Metrics via real-time metric collection and analysis.

#### Common Mistakes/Follow-Ups

A common mistake might be not fully understanding how Datadog integrates different types of data sources or the importance of each pillar in a holistic observability strategy. Follow-ups could include asking about best practices for implementing these pillars, specific use cases, or comparisons with other tools.

**Comparison:**

```
Pillar	Definition	Datadog Approach
Tracing	Following requests through a system.	Supports distributed tracing with OpenTelemetry.
Logging	Collecting application logs for analysis.	Real-time log management and search capabilities.
Metrics	Numerical values indicating system performance.	Metric collection, aggregation, and visualization tools.
```

**Code:**

```python
# Example Python code snippet to collect metrics using OpenTelemetry (OTel)
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.metrics import MeterProvider

trace.set_tracer_provider(TracerProvider())
meter = MeterProvider()

tracer = trace.get_tracer("my_library")

with tracer.start_as_current_span("example_span"):
    # Your application logic here
    pass
```

**Example input / output:**

```text
Input:
- Logs from auth-service and web-app services.
- Metrics for response time and error rate.

Output:
- Traces correlated across services.
- Real-time log search results.
- Customizable dashboards showing metric trends and alerts.
```

**Data:**

```json
{
  "logs": [
    {
      "timestamp": "2023-10-05T09:00:00Z",
      "message": "User login successful",
      "service": "auth-service"
    },
    {
      "timestamp": "2023-10-05T09:01:00Z",
      "message": "API request received - POST /api/v1/user",
      "service": "web-app"
    }
  ],
  "metrics": [
    {
      "name": "response_time",
      "value": 234,
      "unit": "ms",
      "labels": {"service": "auth-service"}
    },
    {
      "name": "error_rate",
      "value": 0.05,
      "unit": "%",
      "labels": {"service": "web-app"}
    }
  ]
}
```

**Flow:**

1. Tracing
2. OpenTelemetry
3. Collect trace data
4. Distribute traces acrossservices
5. Logging
6. Real-time log management
7. Ingest logs from varioussources
8. Search and filter logs
9. Metrics
10. Metric collection and analysis
11. Collect metrics from apps,infra, and DBs
12. Aggregate metrics forvisualization
13. Dashboards
14. Customize dashboards

