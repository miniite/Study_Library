---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up a dashboard that lets you attribute a customer-reported slowness to a specific route, upstream, region, and time window within minutes, without exploding metric cardinality?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Connection Reuse and Client Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge effectively, we need to design an API Gateway that can capture detailed metrics and correlate them with customer-reported slowness issues in real-time. The core idea is to leverage a structured approach to collect and aggregate performance data at various levels—routes, upstream services, regions, and time windows—to pinpoint the source of latency quickly.

#### Triage
Understanding the Question

The question asks us to set up a dashboard that helps identify the specific cause of customer-reported slowness. The key aspects are:

Real-time response: We need metrics within minutes.
Granularity: Metrics should be detailed by route, upstream service, region, and time window.
Cardinality management: Avoiding excessive metric cardinality to prevent data explosion.

#### First-Principles Solution

We will design an API Gateway that:

Captures metrics at different levels (routes, upstream services).
Correlates these metrics with customer-reported issues using a tagging mechanism.
Aggregates and filters data in real-time based on user-defined time windows.
Stores and queries data efficiently to support quick analysis.
Step-by-Step Solution

API Gateway Setup:

Use an API Gateway like NGINX, AWS API Gateway, or Kong to manage incoming requests.
Configure the gateway to capture detailed metrics for each request, including timestamps, response times, and error codes.

Tagging Mechanism:

Introduce tags for routes, upstream services, regions, and time windows.

#### Example
tags could include route_id, upstream_service, region, and time_window.

Real-time Data Collection:

Use a real-time data collection system like Prometheus or CloudWatch to ingest metrics.
Set up alerts for latency thresholds using tools like Alertmanager.

Aggregation and Correlation:

Aggregate metrics by tags to identify performance issues.
Use SQL-like queries in time-series databases (like TimescaleDB) to correlate customer-reported slowness with the captured metrics.

Dashboard Design:

Implement a dashboard using tools like Grafana or Kibana, which can display real-time and historical data.
Configure the dashboard to filter data based on user-defined criteria (e.g., region, time window).

Efficient Storage and Querying:

Use efficient storage solutions that support time-series data, such as InfluxDB or TimescaleDB.
Optimize queries for quick response times by using appropriate indexing techniques.

#### Closing Summary

By setting up an API Gateway with detailed tagging, real-time data collection, and efficient querying, we can ensure that customer-reported slowness issues are quickly attributed to the correct routes, upstream services, regions, and time windows. This approach balances granularity with performance efficiency to provide actionable insights within minutes.

#### Common Mistakes/Follow-Ups
Ignoring Cardinality: Failing to manage metric cardinality can lead to data explosion.
Lack of Real-time Alerts: Not setting up real-time alerts for latency issues can delay issue resolution.
Inefficient Queries: Poorly optimized queries can slow down the dashboard, affecting user experience.

To further refine this solution, consider using machine learning techniques to predict potential bottlenecks and proactively alert on them. Additionally, ensure that the tagging mechanism is flexible enough to accommodate future changes in the API architecture.

**Comparison:**

```
Route	Upstream Service	Region	Time Window Start	Time Window End	Latency (ms)
r123	us101	us-east-1	2023-09-15T08:00:00Z	2023-09-15T09:00:00Z	456
```

**Code:**

```python
# Example configuration for Prometheus metrics collection
from prometheus_client import start_http_server, Counter

route_latency_counter = Counter('route_latency', 'Latency by route and upstream service', ['route_id', 'upstream_service'])

def log_latency(route_id, upstream_service):
    route_latency_counter.labels(route_id=route_id, upstream_service=upstream_service).inc(response_time_ms)
```

**Example input / output:**

```text
Input: Customer reports slow response from route r123 to us101 in region us-east-1.
Output: Dashboard displays detailed metrics for the specified time window, showing latency spikes and potential bottlenecks.
```

**Data:**

```json
{
  "metrics": [
    {
      "route_id": "r123",
      "upstream_service": "us101",
      "region": "us-east-1",
      "time_window": "2023-09-15T08:00:00Z/2023-09-15T09:00:00Z",
      "latency_ms": 456
    }
  ]
}
```

**Flow:**

1. Customer Reports Slowness
2. Tag Requests with Metrics
3. API Gateway Collects Metrics
4. Store Metrics in Time-SeriesDB
5. Query Metrics Real-Time
6. Dynamically UpdateDashboard

