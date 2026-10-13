---
type: "practice"
difficulty: "Deep"
---

# What's your framework, in an interview, for turning a vague 'we want real-time analytics' ask into a concrete batch-vs-stream design decision?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batch vs Streaming Cost Trade-offs](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a vague request like "we want real-time analytics," the first step is to understand the underlying business needs and constraints. This involves probing for details about the data sources, processing requirements, latency goals, and overall scale of the system. From there, you can systematically evaluate whether batch or stream processing better meets those needs.

To turn this into a concrete decision, I would follow these steps:

Clarify Requirements: Understand what "real-time" means in terms of end-to-end latency.
Data Characteristics: Assess if data is arriving incrementally (stream) or in batches.
Processing Needs: Determine the nature and complexity of the analytics required.
Resource Constraints: Consider available compute resources, network bandwidth, and storage limitations.
Cost and Risk: Evaluate trade-offs between simplicity, performance, cost, and reliability.

**Comparison:**

```
Requirement	Decision Factor
Latency	<10 seconds
Data Sources	Incremental
Complexity	High
Resources	Moderate
```

**Code:**

```text
# Example pseudocode for data processing decision
def decide_batch_vs_stream(requirements):
    if requirements['latency'] <= 10:
        return "stream"
    elif requirements['data_sources_incremental']:
        return "stream"
    else:
        return "batch"
```

**Example input / output:**

```text
Given the sample input, the output would be `stream` because both latency and incremental data sources are critical factors.
```

**Data:**

```yaml
# Sample input data for decision-making
requirements:
  latency: 5 # seconds
  data_sources_incremental: true
  processing_complexity: high
  resource_constraints: moderate
```

**Flow:**

1. Yes
2. No
3. Data Sources Incremental
4. No
5. Clarify Requirements
6. Latency <= 10?
7. Stream
8. Batch
9. Stream
10. Batch

