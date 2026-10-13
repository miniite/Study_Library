---
type: "practice"
difficulty: "Deep"
---

# How do you use trace analytics to find which endpoint contributes most to overall p99 latency across a service?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ APM and Trace Analytics](<../../../../TOOLS/Datadog/Datadog_Platform/APM_and_Trace_Analytics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we'll explore how to leverage Datadog's trace analytics to identify which endpoint is contributing the most to overall p99 latency in a complex microservices architecture. This involves understanding how traces work in Datadog and using specific metrics and tools to isolate problematic endpoints.

#### Triaging the Question

The question revolves around pinpointing performance bottlenecks in a distributed system, specifically focusing on high-latency issues (p99) across multiple service endpoints. The interviewer is likely probing your ability to analyze complex data, understand service interactions, and effectively use Datadog’s tools to troubleshoot latency.

#### First-Principles Solution

Understanding Trace Analytics in Datadog:

A trace in Datadog represents a single request as it moves through the entire system from the client to the backend services.
Each service endpoint contributes segments of this trace, and by analyzing these segments, you can identify which part of the process is causing delays.

Metrics and Filters:

We need metrics like latency or duration at each endpoint.
Use filters (service:) to focus on specific services or endpoints.
Apply aggregation functions (sum, max) over time to understand latency patterns.

Using Datadog's Tracing and Metrics:

Retrieve traces for a representative period (e.g., last 24 hours).
Filter these traces by the service you are interested in.
Use Datadog's APM (Application Performance Monitoring) feature to visualize trace segments and their latencies.

Analyzing Trace Data:

Identify the longest segments that contribute significantly to p99 latency.
Look for commonalities among these segments, such as specific API calls or database queries.
Use Datadog's histogram view to get a detailed breakdown of latency distribution.

Isolating Problematic Endpoints:

Compare the latencies between different endpoints within your service.
Identify which endpoint has the highest p99 latency and drill down into its trace segments.
Check for any common patterns or issues, such as database connection timeouts or slow network requests.

#### Closing Summary
+ Common Mistakes

By following these steps, you can effectively use Datadog's trace analytics to identify and resolve performance bottlenecks. Common mistakes include:

Not using appropriate filters, leading to irrelevant data.
Failing to drill down into the detailed trace segments for deeper analysis.
Ignoring aggregated metrics like p95 or p99 latency without considering overall service behavior.
Triaging Input

Let's assume we have the following input:

PLAINTEXT
Service: example-service
Time Period: Last 24 hours
Endpoint A Latency: Avg = 10ms, Max = 300ms (p99)
Endpoint B Latency: Avg = 5ms, Max = 150ms (p99)
Triaging Output

By analyzing the trace segments for example-service, we can see that Endpoint A consistently has a higher p99 latency. Further analysis of its traces reveals that a particular database query is taking an unusually long time.

#### Common Mistakes/Follow-Ups
Misinterpreting average vs. max latencies.
Overlooking edge cases where occasional spikes can dominate the overall p99 metric.
Not considering the impact of network latency or external dependencies.

**Comparison:**

```
Service	Endpoint Name	Avg Latency (ms)	Max Latency (ms)
example-service	A	10	300
	B	5	150
```

**Code:**

```python
# Example code to fetch traces from Datadog API
import datadog_api_client
from datadog_api_client.v1.api.tracing_api import TracingApi

api_instance = TracingApi()
response = api_instance.list_traces(
    filter_service="example-service",
    start=24*3600,  # Last 24 hours
)

for trace in response:
    print(trace)
```

**Example input / output:**

```text
Input:
- Service: example-service
- Time Period: Last 24 hours

Output:
- Endpoint A had a max p99 latency of 300ms.
- Endpoint B had a max p99 latency of 150ms.

Conclusion:
Endpoint A is the primary contributor to high p99 latency, likely due to a problematic database query.
```

**Data:**

```json
{
  "service": "example-service",
  "endpoints": [
    {
      "name": "Endpoint A",
      "latency_avg": 10,
      "latency_max": 300
    },
    {
      "name": "Endpoint B",
      "latency_avg": 5,
      "latency_max": 150
    }
  ]
}
```

**Flow:**

1. Longest p99 latency
2. Fetch Traces
3. Filter by Service
4. Analyze Trace Segments
5. Identify Problematic Endpoint
6. Drill Down into Details

