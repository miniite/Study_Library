---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide the right evaluation delay for a metric that arrives late from a batch ingestion source?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Metrics vs Logs Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Metrics_vs_Logs_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide the right evaluation delay for a metric arriving late from a batch ingestion source in Datadog, we need to understand the trade-offs between accuracy and timeliness, while also considering the nature of the data and the specific use case. The evaluation delay is essentially the time interval after which you want to start aggregating or evaluating metrics that have been ingested in batches.

#### Triaging the Question

The key aspects Datadog probes here are:

Data Latency: Understanding how much latency is acceptable for your data.
Performance Impact: Evaluating how different evaluation delays affect system performance and resource usage.
Use Case Specifics: Considering the criticality of timely insights versus historical accuracy.

#### First Principles Solution

The evaluation delay should be set based on a few fundamental principles:

Latency Tolerance: The maximum acceptable latency for your use case.
Ingestion Frequency: The rate at which data is being ingested in batches.
Aggregation Strategy: How you plan to aggregate or evaluate the batched metrics.
Step-by-Step Solution

Determine Latency Tolerance:

Identify the point where late data becomes actionable. For example, if your use case requires real-time visibility, a 5-minute delay might be unacceptable.

#### Cons
ider using Datadog’s dynamic evaluation features to handle varying latencies gracefully.

Ingestion Frequency Analysis:

Understand how often new batches are arriving and their size. If the ingestion is frequent (e.g., every few minutes), setting an evaluation delay of 5-10 minutes might make sense.
For less frequent ingestion (e.g., hourly batches), a delay of 30-60 minutes could be more appropriate.

Aggregation Strategy:

Decide how you want to aggregate the data. For example, if you are doing simple counts or averages, a shorter evaluation delay might suffice.
If more complex aggregations like percentile calculations are required, a longer delay may be necessary to ensure all data points are included.

Experimentation and Monitoring:

Set up initial evaluations with conservative delays and monitor the impact on performance and accuracy.
Fine-tune based on observed results and user feedback.

#### Closing Summary

In summary, choosing the right evaluation delay involves balancing the acceptable latency for your use case against the performance and accuracy requirements. By understanding the specific needs of your system and experimenting iteratively, you can find an optimal balance that meets both timeliness and accuracy goals.

#### Common Mistakes
and Follow-Ups
Ignoring Data Latency: Often, teams set evaluation delays too low, leading to a loss in accuracy.
Over-Evaluating: Setting overly long delays can lead to performance issues if evaluations are too infrequent.
Not Considering Use Case Specifics: Different use cases may have vastly different requirements for latency and accuracy.

#### Follow-ups
might include:

How do you handle unexpected spikes in data ingestion?
What metrics would you use to validate the chosen evaluation delay?

**Comparison:**

```
Batch Size	Evaluation Delay (s)
100	60
150	180
200	300
```

**Code:**

```python
# Example Python code snippet to simulate batch processing with a configurable evaluation delay
import time

def process_batches(batch_size, delay_seconds):
    # Simulate batches arriving every 5 minutes with a size of `batch_size`
    for i in range(10):  # Process 10 batches for demonstration
        print(f"Processing batch {i+1} with {batch_size} data points")
        time.sleep(delay_seconds)  # Simulate delay

process_batches(batch_size=100, delay_seconds=300)  # Example: 5 minutes delay
```

**Example input / output:**

```text
| Batch Size | Timestamp         | Evaluation Delay (s) |
|------------|------------------|---------------------|
| 100        | 2023-10-01T09:00Z | 60                  |
| 150        | 2023-10-01T09:10Z | 180                 |
| 200        | 2023-10-01T09:20Z | 300                 |

For the above batches, an evaluation delay of 60 seconds is chosen for the first batch to process it quickly. The subsequent batches are delayed by 180 and 300 seconds to account for increasing data size.
```

**Data:**

```json
{
    "batches": [
        {"size": 100, "timestamp": "2023-10-01T09:00Z"},
        {"size": 150, "timestamp": "2023-10-01T09:10Z"},
        {"size": 200, "timestamp": "2023-10-01T09:20Z"}
    ]
}
```

**Flow:**

1. Data Ingestion
2. Batch Size
3. Short Delay, e.g., <60s
4. Long Delay, e.g., >120s
5. Evaluate & Aggregate Quickly
6. Evaluate & Aggregate Slowly
7. Monitor Performance
8. Adjust Evaluation Delay asNeeded
9. Determine Latency Tolerance
10. Determine Latency Tolerance
11. Finalize Evaluation Strategy

