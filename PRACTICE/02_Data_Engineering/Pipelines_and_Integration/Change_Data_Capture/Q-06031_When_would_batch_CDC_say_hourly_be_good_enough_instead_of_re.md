---
type: "practice"
difficulty: "Basic"
---

# When would batch CDC (say hourly) be good enough instead of real-time streaming?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ AWS DMS vs Debezium](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/AWS_DMS_vs_Debezium.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Batch Change Data Capture (CDC) is often preferred over real-time streaming in scenarios where data freshness requirements are not stringent, processing latency can tolerate hourly updates, and the cost of maintaining a real-time system outweighs the benefits.

When considering whether batch CDC (e.g., hourly) would be sufficient instead of real-time streaming, we need to evaluate several factors: data freshness requirements, expected volume of changes, processing latency tolerance, and overall cost. Let’s break down these considerations:

Data Freshness Requirements: If the application can tolerate a delay in processing updates, batch CDC might be adequate.
Volume of Changes: Real-time systems are more suitable for high-frequency data changes, whereas hourly batches can handle lower change volumes efficiently.
Processing Latency Tolerance: In cases where timely decisions based on recent data are not critical, batch processing can provide a simpler and less resource-intensive solution.
Cost Considerations: Real-time streaming solutions often come with higher operational costs due to the continuous monitoring and handling of streams. Batch processes might be more cost-effective for applications that do not require immediate updates.

To illustrate this concept, let's consider an example where we have a customer management system. We need to update reports on customer demographics hourly rather than continuously. In such a scenario, batch CDC would suffice because the delay in processing won't significantly impact our analysis or decision-making processes.

Let’s dive into the detailed reasoning:

Data Freshness Requirements:

If the application can accept an hour's worth of lag in data updates, then hourly batch CDC is appropriate.
For example, if you are generating daily summaries or weekly reports, a one-hour delay might be acceptable and not impact the overall analysis.

Volume of Changes:

In scenarios with low to moderate change frequency, batch processing can be more efficient than real-time streaming.
Real-time systems require constant monitoring and resource allocation even when there are no significant changes.

Processing Latency Tolerance:

If the business logic or analysis does not need immediate updates, a delayed but consistent update schedule is sufficient.
For instance, if your application processes financial transactions daily at night after regular trading hours, batch processing during off-peak times can be highly beneficial.

Cost Considerations:

Real-time systems often have higher costs due to the need for continuous data processing and infrastructure support.
Batch systems require fewer resources when not actively processing, leading to lower operational expenses.

#### Example Scenario

#### Cons
ider a retail company that processes customer purchase events every second. However, they only need to generate daily sales reports by aggregating purchases from each day. In this case:

Real-time Streaming: Would be overkill and expensive, given the low value of immediate updates.
Hourly Batch CDC: Can process all purchases at the end of the hour, providing a near-real-time report without the cost.

#### Closing Summary
Batch Change Data Capture (CDC) is generally sufficient when:

Data freshness requirements are not critical.
Volume of changes can be processed within an acceptable time frame.
Processing latency does not significantly impact the application's functionality.
Cost-effectiveness is a major concern, as real-time systems can be prohibitively expensive for low-frequency data.

#### Common Mistakes
and Follow-Ups:

Mistake: Overlooking the criticality of data freshness in decision-making.

#### Follow-Up
Discussing edge cases where near-real-time updates might still be required despite batch processing. For instance, if immediate feedback is crucial for certain actions like fraud detection.

**Comparison:**

| Timestamp | Event    |
|-----------|----------|
| 2023-10-05T09:01:00Z | purchase |
| 2023-10-05T09:05:00Z | refund   |

**Code:**

```python
# Example Python code to simulate hourly batch CDC

def process_hourly_changes(data):
    # Simulate hourly data processing
    print("Processing changes from 09:00 to 10:00")
    # Logic for aggregating or transforming the data
    # Output processed results at the end of the hour
    pass

# Example input data schema
input_data = [
    {"timestamp": "2023-10-05T09:01:00Z", "event": "purchase"},
    {"timestamp": "2023-10-05T09:05:00Z", "event": "refund"}
]

process_hourly_changes(input_data)
```

**Example input / output:**

```text
Input:
[
    {"timestamp": "2023-10-05T09:01:00Z", "event": "purchase"},
    {"timestamp": "2023-10-05T09:05:00Z", "event": "refund"}
]

Output:
Processing changes from 09:00 to 10:00
```

**Data:**

```json
{
  "input_data": [
    { "timestamp": "2023-10-05T09:01:00Z", "event": "purchase" },
    { "timestamp": "2023-10-05T09:05:00Z", "event": "refund" }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Start Hourly Batch Processing
4. Is Data Freshness Critical?
5. Process Data Every Hour
6. Real-Time Streaming Required
7. Generate Reports/Analytics
8. Monitor and ProcessReal-Time Events

