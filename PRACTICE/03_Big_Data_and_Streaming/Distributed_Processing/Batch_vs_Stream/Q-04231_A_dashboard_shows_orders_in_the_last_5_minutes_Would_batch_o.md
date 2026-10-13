---
type: "practice"
difficulty: "Basic"
---

# A dashboard shows 'orders in the last 5 minutes'. Would batch or stream be more appropriate and why?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Batch vs Streaming Cost Trade-offs](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering a dashboard that needs to display "orders in the last 5 minutes," we must decide between using batch processing and stream processing, based on their characteristics and requirements.

Batch processing involves collecting data over a period of time (a batch) before performing analysis. Stream processing, on the other hand, analyzes data as it arrives in real-time.

For this scenario, stream processing would be more appropriate for several reasons:

Real-Time Nature: The dashboard needs to show the latest orders continuously.
Latency Sensitivity: Users expect the most up-to-date information without significant delays.
Scalability: Stream processing can handle large volumes of data arriving in real-time, which is crucial if the number of orders increases.

Let's break this down from first principles:

#### Triage
The question asks whether batch or stream processing should be used for a dashboard displaying "orders in the last 5 minutes."
Key Considerations:
Real-time updates vs. periodic aggregations.
Data volume and frequency of incoming data.
Latency requirements.

#### Solution

Understanding Batch Processing:

Data Collection: Collects a full batch (e.g., all orders received in the last hour).
Processing Time: Analysis is performed on the entire batch, which can be delayed.

#### Example
If you aggregate orders every hour and then show them on a dashboard once per hour, this would be batch processing.

Understanding Stream Processing:

Continuous Data Ingestion: Processes data as it arrives, without waiting for a full batch.
Latency: Can provide near real-time results since no waiting period is required.

#### Example
If you process each order as soon as it comes in and update the dashboard immediately, this would be stream processing.

Why Stream Processing is More Suitable:

Real-Time Updates: The requirement to show "orders in the last 5 minutes" necessitates real-time updates.
Low Latency: Stream processing can provide low-latency results, ensuring users see the most recent data.
Scalability: As the number of orders increases, stream processing can handle the load more effectively.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: For a dashboard showing "orders in the last 5 minutes," stream processing is the better choice due to its real-time nature and ability to provide low-latency updates.

#### Common Mistakes
Using batch processing when real-time data freshness is critical.
Overlooking the scalability issues of batch processing with high-frequency data streams.
Common Follow-Ups
What happens if the number of orders per minute increases significantly?
How can we ensure data consistency in a stream processing setup?

**Comparison:**

```
Batch vs Stream	Batch Processing	Stream Processing
Data Collection	Full batch first	Continuous stream
Processing Time	Delayed analysis	Immediate processing
Latency	Higher	Lower
Scalability	Limited to batch size	Handles large volumes efficiently
```

**Code:**

```python
# Example Python pseudo-code for real-time order processing

def process_order(order):
    # Process each incoming order (real-time)
    print(f"Order processed: {order}")

# Simulate stream of orders
for i in range(100):
    current_time = datetime.now()
    order = {"id": i, "timestamp": current_time}
    process_order(order)
```

**Example input / output:**

```text
Input (Simulated Order Stream):
{
  "id": 4,
  "timestamp": "2023-10-01T12:15:00Z"
}

Output (Immediate Processing Response):
Order processed: {"id": 4, "timestamp": "2023-10-01T12:15:00Z"}
```

**Data:**

```json
{
  "orders": [
    {"id": 1, "timestamp": "2023-10-01T12:00:00Z"},
    {"id": 2, "timestamp": "2023-10-01T12:05:00Z"},
    {"id": 3, "timestamp": "2023-10-01T12:10:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. No
6. Simulated Order Stream
7. Process Each Order?
8. Immediate ProcessingResponse
9. Collect Batch Data
10. Perform Analysis
11. Continue Collecting

