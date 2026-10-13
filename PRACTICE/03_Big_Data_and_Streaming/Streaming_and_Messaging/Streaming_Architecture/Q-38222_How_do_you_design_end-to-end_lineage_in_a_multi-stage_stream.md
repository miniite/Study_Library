---
type: "practice"
difficulty: "Deep"
---

# How do you design end-to-end lineage in a multi-stage streaming anomaly pipeline so when a bad alert fires you can trace it back to the exact source metric and transformation? What does that cost?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an end-to-end lineage in a multi-stage streaming anomaly detection pipeline, we need to ensure every stage of processing is meticulously tracked from raw metrics to final alerts. This involves implementing comprehensive metadata management and leveraging robust tracing mechanisms. The key idea here is that each transformation and filtering step should maintain a clear audit trail.

The primary challenge is ensuring traceability when an anomaly alert fires, enabling us to backtrack the exact source metric or intermediate data state. We must also consider the overhead and cost implications of such detailed lineage tracking in our overall system design.

To achieve this, we will:

Implement structured metadata tagging at each processing step.
Use distributed tracing tools like Jaeger or OpenTelemetry for end-to-end traceability.
Design a storage layer that can efficiently store and query the lineage data.
Ensure scalability and performance in both data flow and trace retrieval.

Let's dive into the detailed steps to build this system from first principles.

**Comparison:**

```
Step	Metadata Key	Value
Pipeline Step 1	processed_at	2023-10-01T10:05:00Z
Pipeline Step 2	processed_at	2023-10-01T10:10:00Z
Anomaly Detection	detected_at	2023-10-01T10:15:00Z
```

**Code:**

```python
# Example metadata tagging function for processing step
def process_data(data, metadata):
    # Process data
    processed_data = transform_data(data)
    # Tag with metadata
    metadata['processed_at'] = datetime.now()
    return processed_data, metadata

# Example tracing context propagation
class TracingContext:
    def __init__(self, trace_id, span_id):
        self.trace_id = trace_id
        self.span_id = span_id

    def to_dict(self):
        return {'trace_id': self.trace_id, 'span_id': self.span_id}

# Example of propagating tracing context through a pipeline step
def process_with_tracing(data, metadata):
    ctx = TracingContext(trace_id='12345', span_id='67890')
    processed_data, updated_metadata = process_data(data, {**metadata, **ctx.to_dict()})
    return processed_data, updated_metadata

# Example tracing setup in a processing pipeline
def pipeline_step_1(data):
    ctx = TracingContext(trace_id='12345', span_id='67890')
    processed_data, updated_metadata = process_with_tracing(data, {**metadata, **ctx.to_dict()})
    return processed_data

# Example tracing setup in a processing pipeline
def pipeline_step_2(data):
    ctx = TracingContext(trace_id='12345', span_id='67890')
    processed_data, updated_metadata = process_with_tracing(data, {**metadata, **ctx.to_dict()})
    return processed_data

# Example tracing setup in a processing pipeline
def anomaly_detection(data):
    ctx = TracingContext(trace_id='12345', span_id='67890')
    result = detect_anomaly(processed_data)
    if result:
        # Log trace information
        log_anomaly_trace(ctx.trace_id, ctx.span_id, 'Anomaly detected at step 3')
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "data": [1, 2, 3, 4, 5],
  "metadata": {
    "source_metric_id": "metric-001",
    "source_system": "system-A"
  }
}

Output:

JSON
Copy
{
  "trace_id": "12345",
  "span_id": "67890",
  "anomaly_detected": true,
  "steps": [
    {
      "step": "pipeline_step_1",
      "metadata": {
        "processed_at": "2023-10-01T10:05:00Z"
      }
    },
    {
      "step": "pipeline_step_2",
      "metadata": {
        "processed_at": "2023-10-01T10:10:00Z"
      }
    },
    {
      "step": "anomaly_detection",
      "metadata": {
        "detected_at": "2023-10-01T10:15:00Z"
      }
    }
  ]
}
```

**Data:**

```yaml
trace_information:
  - trace_id: "12345"
    span_id: "67890"
    timestamp: "2023-10-01T10:00:00Z"
    steps:
      - step: "pipeline_step_1"
        metadata:
          processed_at: "2023-10-01T10:05:00Z"
      - step: "pipeline_step_2"
        metadata:
          processed_at: "2023-10-01T10:10:00Z"
      - step: "anomaly_detection"
        anomaly_detected: true
```

**Flow:**

1. No
2. Yes
3. Source Metric
4. Pipeline Step 1
5. Pipeline Step 2
6. Anomaly Detection
7. Anomaly Detected?
8. F
9. Trace Anomaly
10. Log Trace Information

