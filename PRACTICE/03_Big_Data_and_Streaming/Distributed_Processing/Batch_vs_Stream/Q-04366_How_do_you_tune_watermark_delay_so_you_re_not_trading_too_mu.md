---
type: "practice"
difficulty: "Deep"
---

# How do you tune watermark delay so you're not trading too much latency for correctness on late data? What data do you look at to set it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Watermarks and State Management](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Watermarks_and_State_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to tune watermark delay in a way that balances latency and correctness, we need to understand the core principles of both batch and stream processing and their trade-offs. Watermarks are used in stream processing to handle late data by ensuring it does not disrupt the processing results. Setting the watermark delay is crucial as it directly impacts the accuracy of your analytics while affecting the overall system's real-time responsiveness.

The key idea here is to find a balance between the window size, watermark generation, and the acceptable level of late data. Here’s a step-by-step approach to determine an optimal watermark delay:

#### Triaging the Question
Understanding Watermarks: In stream processing, watermarks are used to indicate when all events in a batch have arrived for a specific time interval. This is essential for handling late arriving data.
Latency vs Correctness Trade-off: The primary goal is to minimize latency while ensuring that the processed results remain correct and consistent with the current state of the data.

#### Assumptions
We are working in an environment where stream processing is required due to high throughput and real-time requirements.
Late data can still provide valuable insights, but excessive late events can degrade system performance and accuracy.
Step-by-Step Solution from First Principles
Determine the Business Requirements: Understand what level of lateness your application can tolerate. For example, if you're processing financial transactions, a few seconds of delay might be acceptable.
Analyze Late Data Distribution:
Collect data on how often late data occurs and within what timeframe. This helps in understanding typical patterns and anomalies.
Set Initial Watermark Delay: Start with a conservative watermark delay that ensures most events are processed quickly but still accounts for some lateness.
Monitor System Performance:
Use real-time monitoring tools to track the number of late events, processing latency, and overall system throughput.
Iterative Adjustment:
Gradually increase or decrease the watermark delay based on observed data. Ensure that increasing the watermark delay does not significantly affect the system's ability to process current data.
Common Data to Monitor
Late Event Count: Track how many events are arriving late over a given period.
Processing Latency: Measure the time taken from receiving an event to its inclusion in the final output.
Watermark Age Distribution: Analyze the age of watermarks to ensure they are not too far behind the current timestamp.

#### Example Scenario

Let's assume you have a stream processing system for online transaction analysis. You notice that 5% of transactions arrive late within a few seconds, and these late events still provide valuable insights. Here’s how you might proceed:

Initial Setting: Set an initial watermark delay of 2-3 seconds.
Monitoring:
Over the next week, collect data on late event counts and processing times.
Adjust the watermark delay to a more optimal value, such as 4 seconds, if it results in fewer late events without increasing latency significantly.

#### Closing Summary

In summary, tuning watermark delay involves balancing between real-time responsiveness and data correctness. By monitoring key performance indicators like late event count and processing latency, you can iteratively adjust the watermark delay to find a sweet spot that meets both business needs and technical constraints.

#### Common Mistakes
Overly Aggressive Watermark Delay: Setting it too low might result in excessive late events, compromising accuracy.
Ignoring Late Event Patterns: Failing to analyze when late data typically occurs can lead to suboptimal tuning.

#### Follow-ups
How do different watermark delay settings affect system throughput and resource utilization?
Are there specific use cases where a constant watermark is more suitable than dynamic adjustment?

**Comparison:**

| Time Interval | Late Events |
|---------------|-------------|
| 0-2s          | 98%         |
| 2-10s         | 1.5%        |
| >10s          | 0.5%        |

**Code:**

```python
# Example code for monitoring watermark age distribution
def monitor_watermark_age_distribution(data):
    age_distribution = {}
    for event in data:
        timestamp = event['timestamp']
        watermark_age = abs(timestamp - current_time())
        if watermark_age not in age_distribution:
            age_distribution[watermark_age] = 1
        else:
            age_distribution[watermark_age] += 1
    return age_distribution

# Example monitoring function
def monitor_processing_latency(data):
    latency_data = []
    for event in data:
        start_time = event['received_time']
        end_time = event['processed_time']
        latency = (end_time - start_time).total_seconds()
        latency_data.append(latency)
    return latency_data
```

**Example input / output:**

```text
Input:
[
    {"timestamp": 1634089520, "received_time": "2021-10-10T17:05:20Z", "processed_time": "2021-10-10T17:05:25Z"},
    {"timestamp": 1634089530, "received_time": "2021-10-10T17:05:30Z", "processed_time": "2021-10-10T17:05:33Z"},
    {"timestamp": 1634089540, "received_time": "2021-10-10T17:05:40Z", "processed_time": "2021-10-10T17:05:44Z"}
]

Monitoring Output:
{
    "Watermark Age Distribution": {
        5: 1,
        8: 1,
        14: 1
    },
    "Processing Latency": [5.0, 3.0, 4.0]
}
```

**Data:**

```json
{
  "data": [
    {"timestamp": 1634089520, "received_time": "2021-10-10T17:05:20Z", "processed_time": "2021-10-10T17:05:25Z"},
    {"timestamp": 1634089530, "received_time": "2021-10-10T17:05:30Z", "processed_time": "2021-10-10T17:05:33Z"},
    {"timestamp": 1634089540, "received_time": "2021-10-10T17:05:40Z", "processed_time": "2021-10-10T17:05:44Z"}
  ]
}
```

**Flow:**

1. Late Events Increase
2. Lateness within acceptablerange
3. Initial Watermark Delay
4. Monitor System Performance
5. Increase Watermark Delay
6. Sustain Current Settings
7. Iterate and Optimize

