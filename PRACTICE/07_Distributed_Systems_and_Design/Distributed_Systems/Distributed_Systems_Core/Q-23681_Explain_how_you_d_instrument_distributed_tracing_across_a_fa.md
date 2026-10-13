---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd instrument distributed tracing across a fan-out inference request (gateway to feature store to model to post-processor) to find which hop in a 6-service path is your latency culprit. What's the sampling trade-off at high QPS?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Tracing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Tracing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Distributed tracing is essential for understanding and optimizing the performance of microservices architectures, especially in complex pipelines like inference requests. In a scenario where we have a fan-out request from a gateway to multiple services (feature store, model, post-processor), distributed tracing can help identify which service introduces the most latency. To achieve this, we need to instrument each service with a tracing library that captures and propagates trace IDs and spans. Here’s how you would approach it:

#### Overview
We will use a distributed tracing system like OpenTelemetry to capture trace information at every hop in the request path. By analyzing these traces, we can pinpoint which service is responsible for the latency.

#### Triage
The core issue is identifying the bottleneck in the pipeline. The interviewer is probing your understanding of distributed tracing concepts and how you would apply them in a high-QPS environment. Key assumptions include that each service has been instrumented with a tracing library, and we have access to a monitoring system capable of analyzing these traces.

#### First-Principles Solution

Step 1: Instrument all services with a distributed tracing library (like OpenTelemetry). Ensure that every function or microservice captures the start and end times for each operation.
Step 2: Propagate trace IDs through the request path using headers. This ensures that related operations can be correlated even if they occur across different services.
Step 3: Implement a tracing backend (like Jaeger) to collect and store these traces. Configure it to handle high QPS by optimizing storage and query performance.
Step 4: Develop dashboards and alerts in your monitoring system to visualize latency distribution, identifying the service with the highest contribution to overall latency.

By following this approach, we can effectively diagnose where the bottleneck lies in a complex pipeline like the one described.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Closing: In summary, instrumenting each microservice with distributed tracing tools allows us to trace requests as they pass through multiple services. By analyzing these traces, we can identify which service is introducing latency.

#### Common Mistakes
One common mistake is not properly propagating trace IDs across services, leading to disjointed trace data that makes it harder to correlate operations. Another mistake is underestimating the storage and query requirements for high QPS environments.

#### Follow-Ups
The interviewer might ask about specific tools or libraries you would use, how you would handle sampling at high QPS, or strategies for optimizing the tracing system.

**Comparison:**

```
Service	Start Time	End Time	Duration
Gateway	10:00:00	10:00:05	5ms
Feature Store	10:00:05	10:00:10	5ms
Model	10:00:10	10:00:20	10ms
Post-Processor	10:00:20	10:00:30	10ms
```

**Code:**

```python
# Example of setting up a span in OpenTelemetry

from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor, ConsoleExport

# Initialize tracer provider and exporter
provider = TracerProvider()
processor = BatchSpanProcessor(ConsoleExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

tracer = trace.get_tracer(__name__)

def process_request():
    with tracer.start_as_current_span("gateway"):
        # Gateway logic here
        pass

    with tracer.start_as_current_span("feature_store"):
        # Feature store logic here
        pass

    with tracer.start_as_current_span("model"):
        # Model inference logic here
        pass

    with tracer.start_as_current_span("post_processor"):
        # Post-processing logic here
        pass
```

**Example input / output:**

```text
Input: HTTP request to the gateway service

Output:
- Traces recorded in each microservice (gateway, feature_store, model, post_processor)
- Tracing backend receives and stores traces
- Monitoring dashboard displays latency insights
```

**Data:**

```json
{
  "trace_id": "e4568790f3b44a1c87295df3b8d385af",
  "span_ids": [
    "1234567890abcdef",
    "fedcba9876543210",
    "109876543210feda"
  ]
}
```

**Flow:**

1. Gateway
2. Feature Store
3. Model
4. Post-Processor

