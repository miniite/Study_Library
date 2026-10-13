---
type: "practice"
difficulty: "Expert"
---

# How would you make explicit and queryable the dependency graph of a system-of-systems, so you can answer 'if service X degrades, which customer journeys break' before an incident rather than during one?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Correlated Failure Prevention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Correlated_Failure_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of making explicit and queryable the dependency graph of a system-of-systems to proactively identify affected customer journeys when service X degrades, we need to design a comprehensive monitoring framework that not only captures dependencies but also correlates them with business-critical user flows. This involves several key steps: defining clear metrics for service health, creating a dependency model, implementing dynamic tracing, and building a queryable interface.

Firstly, let's triage the question: The core of this problem is about establishing a robust monitoring system that can help us understand how changes in one part of our system might impact multiple customer journeys. The goal is to detect potential issues before they manifest as actual incidents, enabling proactive mitigation and improved user experience.

We will assume we have access to detailed service health metrics (e.g., response time, error rate), a dependency graph that maps services to their customers' journeys, and the ability to instrument our services with tracing tools. Our solution will consist of several layers: data collection, dependency modeling, dynamic tracing, and query capabilities.

Step 1: Define Metrics for Service Health We need reliable metrics to gauge service health accurately. Commonly used metrics include:

Response Time (average, p95, p99)
Error Rate (percentage of requests that result in errors)
Throughput (number of successful requests per second)

Step 2: Create a Dependency Model The dependency model should map services and their interactions to specific customer journeys. This can be represented as a graph where nodes are services and edges are dependencies between them, with additional metadata indicating the impact on user journeys.

Service X
Service Y
Service Z
Customer Journey 1
Customer Journey 2

Step 3: Implement Dynamic Tracing Dynamic tracing involves instrumenting services to capture relevant data during runtime. We can use tools like OpenTelemetry for this purpose, which provides standardized instrumentation and export mechanisms.

Service X
Start Span
Capture Data
End Span
Report Metrics

Step 4: Build a Queryable Interface Finally, we need to build an interface that allows us to query this data effectively. This can be a REST API or a more sophisticated GraphQL endpoint.

Monitoring API
Query Dependency Graph
Retrieve Metrics

By integrating these components, we create a system that can predict potential issues and proactively notify us about which customer journeys might be impacted by service degradations. This helps in maintaining high availability and satisfaction levels.

**Comparison:**

```
Metric	Definition	Example Value
Response Time	Average time taken for a request to be processed and a response returned.	200ms
Error Rate	Percentage of requests resulting in errors.	1%
Throughput	Number of successful requests per second.	5,000 req/s
```

**Code:**

```python
# Example of using OpenTelemetry for dynamic tracing
from opentelemetry import trace

tracer = trace.get_tracer("service_x")

with tracer.start_as_current_span("capture_data"):
    # Simulate data capture logic
    pass
```

**Example input / output:**

```

Input (Sample Dependency Graph Data):

JSON
Copy
{
  "dependencies": [
    {
      "service": "X",
      "downstream_services": ["Y", "Z"],
      "journeys_impacted": ["Journey1", "Journey2"]
    },
    {
      "service": "Y",
      "downstream_services": ["W", "V"],
      "journeys_impacted": ["Journey3", "Journey4"]
    }
  ]
}

Output (Query Response):

JSON
Copy
{
  "affected_journeys": [
    {"journey": "Journey1", "impact": "High"},
    {"journey": "Journey2", "impact": "Medium"},
    {"journey": "Journey3", "impact": "Low"}
  ]
}
```

**Data:**

```json
{
  "dependencies": [
    {
      "service": "X",
      "downstream_services": ["Y", "Z"],
      "journeys_impacted": ["Journey1", "Journey2"]
    }
  ]
}
```

**Flow:**

1. Service X
2. Start Span
3. Capture Data
4. End Span
5. Report Metrics

