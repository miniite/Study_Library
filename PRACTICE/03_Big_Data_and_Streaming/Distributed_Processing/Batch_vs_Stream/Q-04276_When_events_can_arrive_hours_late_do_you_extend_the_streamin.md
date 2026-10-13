---
type: "practice"
difficulty: "Deep"
---

# When events can arrive hours late, do you extend the streaming window or push corrections through a batch job? Walk the decision.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batch vs Streaming Cost Trade-offs](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with delayed events in stream processing systems, one must carefully consider whether to extend the streaming window or handle corrections via a batch job. This decision hinges on factors such as event delay tolerance, system complexity, and the desired trade-offs between latency and accuracy.

To start, let’s triage the question: The core issue is understanding how to manage delayed events within stream processing pipelines effectively. We’ll evaluate the implications of extending streaming windows versus using a batch job for corrections, considering both technical and operational aspects.

Firstly, consider the nature of your system. If event delays are infrequent and within an acceptable range (e.g., minutes), extending the streaming window might be sufficient. This approach simplifies real-time processing by allowing late-arriving events to be processed with existing stream records, reducing the need for complex state management or additional data storage.

However, if frequent large delays occur—hours in your case—an extended streaming window may become impractical due to increased complexity and potential resource constraints. In such scenarios, a batch job becomes more viable. A batch job can process late-arriving events at periodic intervals, ensuring that all events are eventually accounted for without bloating the streaming pipeline.

Let’s break down both approaches in detail:

Streaming Window Extension

#### Pros

Simpler Architecture: Extending the window keeps the system stateless and relies on existing processing logic.
Real-Time Processing: Events can be processed as they arrive, maintaining low latency for most cases.
Scalability: Existing stream processors handle the data naturally without requiring significant changes.

#### Cons

Complexity: Managing late-arriving events within a streaming window requires careful state management and potentially complex backpressure handling.
Resource Overhead: Increased memory usage and computational load for processing older events, which can affect performance.
Latency Issues: If the delay tolerance is too high, real-time insights may be compromised.
Batch Job Approach

#### Pros

Simplified State Management: No need to manage complex state within the streaming pipeline; just store late-arriving events for a batch process.
Resource Efficiency: Processed in batches, reducing real-time resource demands and allowing for optimized use of computational resources.
Flexibility: Can be designed with greater tolerance for delay without affecting ongoing stream processing.

#### Cons

Increased Delay: There will be an additional delay between when the event arrives and when it is processed by the batch job, which might not be acceptable in all scenarios.
Complexity Introduced: Additional layers of logic are needed to handle late-arriving events properly.
Operational Overhead: Requires managing a separate process for corrections, adding operational complexity.
Practical Example

Imagine we have an e-commerce system where product inventory levels need to be updated in near real-time. If a purchase event arrives two hours late, it’s critical that the inventory level reflects this transaction accurately. Here, extending the streaming window might not suffice due to the impact on other transactions and state management complexity.

Instead, implementing a batch job that processes all late-arriving events every hour would ensure that no transaction is missed, while keeping real-time processing relatively simple and efficient.

#### Conclusion

Extending the streaming window or handling corrections through a batch job both have their merits. The decision depends on specific use cases, delay tolerances, system architecture, and operational requirements. In your scenario with hourly delays, a batch job appears more practical due to its ability to manage complex state issues and ensure full event processing without overwhelming real-time systems.

#### Common mistakes
include underestimating the impact of delayed events on streaming windows or over-simplifying the need for batch corrections, which can lead to suboptimal performance and system complexity.

**Comparison:**

| Event ID | Timestamp            | Processed |
|----------|----------------------|-----------|
| 1        | 2023-10-05T14:00:00Z | Yes       |
| 2        | 2023-10-05T16:00:00Z | No        |
| 3        | 2023-10-05T14:30:00Z | Yes       |

**Code:**

```python
# Example of a simple stream processing extension logic in Python

def process_event(event):
    if event.timestamp > current_timestamp - window_size:
        # Process event with existing state
        pass
    else:
        # Store for batch job processing
        late_arriving_events.append(event)

# Example of a batch correction job
def batch_correct_events():
    global late_arriving_events
    for event in late_arriving_events:
        # Apply corrections to the system state
        apply_correction(event)
    late_arriving_events = []
```

**Example input / output:**

```text
Input:
[{"id": 1, "timestamp": "2023-10-05T14:00:00Z"}, {"id": 2, "timestamp": "2023-10-05T16:00:00Z"}, {"id": 3, "timestamp": "2023-10-05T14:30:00Z"}]

Output:
late_arriving_events = [{"id": 2, "timestamp": "2023-10-05T16:00:00Z"}]
```

**Data:**

```json
{
  "events": [
    {"id": 1, "timestamp": 2023-10-05T14:00:00Z},
    {"id": 2, "timestamp": 2023-10-05T16:00:00Z}, // Late by 2 hours
    {"id": 3, "timestamp": 2023-10-05T14:30:00Z}
  ],
  "late_arriving_events": []
}
```

**Flow:**

1. Yes
2. No
3. Event Arrival
4. Is Timestamp within Window?
5. Process Event
6. Store for Batch Job
7. Batch Correction Job
8. Apply Corrections

