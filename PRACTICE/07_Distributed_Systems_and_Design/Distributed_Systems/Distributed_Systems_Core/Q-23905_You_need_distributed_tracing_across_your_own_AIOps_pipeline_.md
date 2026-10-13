---
type: "practice"
difficulty: "Deep"
---

# You need distributed tracing across your own AIOps pipeline (ingest, enrich, detect, correlate, remediate) to find which stage adds latency to time-to-detect. Explain context propagation across async queue hops and the sampling trade-off at high telemetry volume.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Tracing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Tracing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this distributed tracing requirement in an AIOps pipeline, we need to ensure that every part of the system can trace its operations back to their origins, which is crucial for identifying where delays occur and pinpointing the root cause. Specifically, across our ingest, enrich, detect, correlate, and remediate stages, context propagation must be seamless. We'll focus on how asynchronous queue hops affect this process and discuss the sampling trade-off when dealing with high telemetry volumes.

Context propagation in distributed systems is essential for maintaining a coherent trace of events as they move through different components. Each stage of our AIOps pipeline needs to pass along key metadata about the request, such as trace IDs, spans, service names, timestamps, etc., to ensure that these details are preserved even when asynchronous processing occurs.

At its core, context propagation involves setting up a consistent way for each microservice or component in the pipeline to share this crucial information. For example, using HTTP headers (like X-Request-ID), custom attributes, or inter-process communication mechanisms can achieve this. The key challenge is ensuring that every part of our system is aware and correctly handles these metadata.

Now let's consider the asynchronous queue hops specifically. In a scenario where data is ingested into an async queue, processed by multiple services, and then outputted to another queue (or directly handled), context propagation must remain intact. This often involves using middleware that intercepts messages as they enter and exit queues or using language-specific libraries like OpenTracing or OpenTelemetry that abstract the complexity.

For high telemetry volumes, traditional tracing can become a bottleneck due to the sheer volume of data generated. Here, we need to consider sampling strategies to balance between detailed diagnostics and system performance. There are three main sampling methods:

Rate Sampling: Only trace requests based on a fixed or dynamic rate.
Probability Sampling: Trace each request with a certain probability (e.g., 0.5%).
Reservoir Sampling: Select a fixed number of samples from the stream.

Probability and reservoir sampling are particularly useful in high-velocity scenarios, as they allow us to gather sufficient insights while minimizing overhead. However, the choice should be guided by the specific needs of our AIOps pipeline—what delays we care about most, and where performance is critical versus diagnostic precision.

In conclusion, context propagation must be robust across all stages of an asynchronous system, particularly when dealing with high telemetry volumes. Strategic sampling can help manage this complexity without sacrificing the quality of diagnostics.

**Comparison:**

| Method        | Description                                                                                         | Pros                     | Cons                      |
|---------------|-----------------------------------------------------------------------------------------------------|--------------------------|----------------------------|
| Rate Sampling | Only trace a subset of requests based on a fixed or dynamic rate                                    | Simple to implement       | May miss important cases  |
| Probability   | Trace each request with a certain probability (e.g., 0.5%)                                          | Balances diagnostics and performance | Requires careful tuning   |
| Reservoir     | Select a fixed number of samples from the stream                                                   | Effective for high volume| Complexity in implementation |

**Code:**

```python
# Example of a simple trace context setup using OpenTelemetry in Python

from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider, SpanKind
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.sdk.resources import Resource
import time

resource = Resource.create({"service.name": "example-service"})
trace_provider = TracerProvider(resource=resource)
batch_exporter = BatchSpanProcessor()  # Replace with actual exporter
trace_provider.add_span_processor(batch_exporter)

# Create a tracer from the provider
tracer = trace.get_tracer(__name__)

def process_request():
    with tracer.start_as_current_span("process_request", kind=SpanKind.SERVER):
        time.sleep(1)  # Simulate processing

def fetch_data_from_queue(queue):
    with tracer.start_as_current_span("fetch_data_from_queue", kind=SpanKind.CLIENT):
        data = queue.get()  # Simulate getting from async queue
        process_request()
        return data

# Example usage
if __name__ == "__main__":
    fetch_data_from_queue(None)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Fetch data from queue and process request. | Trace spans recorded with context propagation, sampling applied based on chosen strategy. |

**Data:**

```json
{
  "trace": {
    "id": "00-4d26e71b9c345e8a-3f5d534ec001b514-00",
    "span": [
      {
        "name": "process_request",
        "kind": "server",
        "attributes": {
          "service.name": "example-service"
        },
        "start_time": 1682790339.123
      },
      {
        "name": "fetch_data_from_queue",
        "kind": "client",
        "attributes": {
          "service.name": "example-service"
        },
        "start_time": 1682790339.124,
        "child_of": "00-4d26e71b9c345e8a-3f5d534ec001b514-00"
      }
    ]
  }
}
```

**Flow:**

1. Ingest
2. Enrich
3. Detect
4. Correlate
5. Remediate

