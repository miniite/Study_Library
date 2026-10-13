---
type: "practice"
difficulty: "Expert"
---

# Why might moving from Lambda to Kappa change how you maintain denormalized aggregates and what do you give up?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Denormalization in Lakes](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In transitioning from a Lambda architecture to a Kappa (streaming) architecture, maintaining denormalized aggregates becomes more challenging due to the inherent nature of real-time streaming data processing. The core idea is that in a Lambda setup, you handle batch and stream data separately, whereas in Kappa, both types of data are processed through a single pipeline. This change necessitates a different approach to managing denormalized aggregates.

#### Triage

The question probes your understanding of the differences between Lambda and Kappa architectures and how these differences affect the maintenance of denormalized aggregates. Key assumptions include familiarity with batch processing vs. real-time streaming, data consistency models (eventual consistency), and state management strategies.

#### First-Principles Solution

Lambda Architecture Overview: In a Lambda setup, you have separate pipelines for batch and stream data. Batch data is processed periodically using MapReduce or similar techniques, while stream data is handled in near real-time using frameworks like Apache Storm or Kafka Streams.

Kappa Architecture Overview: Kappa (or Event-Driven) architecture processes both batch and stream data through a single pipeline, leveraging techniques such as stateful processing with windowing or sessionization to maintain consistency.

Maintaining Denormalized Aggregates in Lambda:

Periodic Processing: Batch jobs are run periodically to recompute aggregates.

#### Cons
istency Guarantees: Ensuring consistent states across batch and stream data can be managed using techniques like watermarking or sessionization.
State Management: Maintaining state is more straightforward due to periodic snapshots.

Maintaining Denormalized Aggregates in Kappa:

Real-Time Processing: Aggregations need to be updated in near real-time as new events arrive.
Eventual Consistency: The system must handle eventual consistency, which can lead to temporary discrepancies.
State Management: Managing state becomes more complex due to the continuous nature of data flow.

#### Trade-offs

Complexity: Kappa introduces complexity in managing state and ensuring data consistency.
Performance: Real-time processing may require more sophisticated techniques like windowing or sessionization, which can impact performance.
Cost: Continuous processing can be resource-intensive compared to batch processing.

#### Closing Summary

In moving from Lambda to Kappa, the maintenance of denormalized aggregates requires a shift towards real-time state management and handling eventual consistency. This change comes with increased complexity but also potential for lower latency and more efficient use of resources in certain scenarios.

#### Common Mistakes/Follow-Ups
Underestimating Complexity: Forgetting that Kappa architecture introduces significant complexities, such as managing state across multiple events.
Ignoring Performance Impact: Assuming that the performance overhead of real-time processing is negligible without careful consideration.
Overlooking Consistency Models: Failing to address how eventual consistency will be managed in a streaming environment.

**Comparison:**

| Time Window | Elements Processed | State Update |
|-------------|--------------------|--------------|
| [00:00, 01:00) | [{"id": 1, "timestamp": "2023-01-01T00:00:01Z", "value": 10}, {"id": 2, "timestamp": "2023-01-01T00:00:02Z", "value": 20}] | sum=30, count=2 |

**Code:**

```python
# Example code for state management in Kappa architecture using Apache Beam
def process_element(element, state):
    # Process element and update state if necessary
    pass

# Define the pipeline with windowing and sessionization
pipeline = beam.Pipeline(options=beam.pipelineOptions)
windowed_elements = (
    pipeline
    | 'Read from source' >> ReadFromSource()
    | 'Window elements' >> beam.WindowInto(window.FixedWindows(60))
    | 'Process elements' >> beam.ParDo(process_element, state=state)
)

pipeline.run().wait_until_finish()
```

**Example input / output:**

```text
Input:
[{"id": 1, "timestamp": "2023-01-01T00:00:01Z", "value": 10}, {"id": 2, "timestamp": "2023-01-01T00:00:02Z", "value": 20}]

Output:
{
  "sum": 30,
  "count": 2
}
```

**Data:**

```json
{
  "events": [
    {"id": 1, "timestamp": "2023-01-01T00:00:01Z", "value": 10},
    {"id": 2, "timestamp": "2023-01-01T00:00:02Z", "value": 20}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Event Stream
4. Is Window Over?
5. Update State
6. Compute Aggregates
7. Persist Aggregates

