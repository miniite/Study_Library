---
type: "practice"
difficulty: "Intermediate"
---

# Compare tumbling, sliding, and session windows for building a real-time GMV dashboard, giving a concrete use for each.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Multi-granularity Windows](<../../../../TOOLS/Apache_Flink/Flink_Core/Multi-granularity_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this answer, we'll compare three windowing types in Apache Flink: tumbling, sliding, and session windows. We’ll discuss their characteristics, suitability for building a real-time GMV (Gross Merchandise Volume) dashboard, and provide concrete use cases for each.

#### Triage

The question aims to understand the differences between these windowing strategies in the context of a real-time streaming application like a GMV dashboard. The key is recognizing how different window types handle incoming data streams and ensuring that they meet the performance and accuracy requirements of the dashboard.

#### Assumptions

The GMV dashboard needs to aggregate financial transactions.
Data arrives in a streaming fashion from multiple sources.
We need to process this data with minimal latency while maintaining accurate calculations.
Detailed Solution
Tumbling Windows

Tumbling windows divide the incoming stream into fixed-size intervals, processing each interval independently. Each window contains exactly the same number of records.

Characteristics:

Fixed and uniform-sized intervals.
Easy to reason about due to their simplicity.
Suitable for scenarios where you need to aggregate data over regular time periods (e.g., every minute).

Use Case: For a GMV dashboard, tumbling windows can be used to calculate the total GMV within each fixed time interval. This is useful when you want to report on financial metrics in a consistent and predictable manner.

#### Example

SQL
SELECT window_start_time, SUM(amount) AS gmv
FROM transactions
GROUP BY TUMBLE(timestamp, INTERVAL 1 MINUTE);
Sliding Windows

Sliding windows also divide the stream into intervals but allow overlapping of these intervals. The size of each interval is fixed, and it slides over the data as new records arrive.

Characteristics:

Overlapping intervals.
More flexible than tumbling windows in terms of processing partially complete windows.
Suitable for scenarios where you need to capture more recent events within a sliding time frame.

Use Case: For a GMV dashboard, sliding windows can be used when you want to analyze the most recent transactions. This is useful if you want to report on the current state of GMV in near real-time, which can help with decision-making based on the latest data.

#### Example

SQL
SELECT window_start_time, SUM(amount) AS gmv
FROM transactions
GROUP BY Tumble(timestamp, INTERVAL 1 MINUTE, INTERVAL 5 SECONDS);
Session Windows

Session windows aggregate records that occur in bursts separated by a defined gap. The size of the session window is determined by a timeout period; if no data arrives within this period, the window is considered closed.

Characteristics:

Variable-sized intervals based on gaps between events.
Suitable for scenarios where you need to process bursts of data followed by quiet periods.

Use Case: For a GMV dashboard, session windows can be used when you want to aggregate transactions that happen within a short burst and then ignore them if no new transaction arrives for some time. This is useful in situations where transactions may not occur frequently but need to be aggregated when they do.

#### Example

SQL
SELECT window_start_time, SUM(amount) AS gmv
FROM transactions
GROUP BY SESSION(timestamp, INTERVAL 5 MINUTES);

#### Closing Summary
+ Common Mistakes/Follow-ups

In summary, the choice of window type depends on the specific requirements of your GMV dashboard. Tumbling windows are best for fixed-time period aggregation, sliding windows provide flexibility with overlapping intervals, and session windows handle bursts of data efficiently.

#### Common Mistakes

Overlooking the importance of latency versus accuracy trade-offs.
Not considering the historical data that might need to be accounted for in real-time dashboards.
Misunderstanding the implications of window overlap and aggregation in sliding and tumbling windows.

For follow-ups, you may want to explore the impact of batch processing on streaming applications, the role of state management in Flink, and how different window types affect resource utilization and performance at scale.

**Comparison:**

```
Window Type	Characteristics	Use Case
Tumbling	Fixed-size intervals	Fixed-time period aggregation, historical data analysis
Sliding	Overlapping intervals	Flexibility with recent events, near real-time updates
Session	Variable-sized based on gaps	Handling bursts of data, ignoring quiet periods
```

**Code:**

```sql
-- Tumbling Window Example
SELECT window_start_time, SUM(amount) AS gmv
FROM transactions
GROUP BY TUMBLE(timestamp, INTERVAL 1 MINUTE);

-- Sliding Window Example
SELECT window_start_time, SUM(amount) AS gmv
FROM transactions
GROUP BY Tumble(timestamp, INTERVAL 1 MINUTE, INTERVAL 5 SECONDS);

-- Session Window Example
SELECT window_start_time, SUM(amount) AS gmv
FROM transactions
GROUP BY SESSION(timestamp, INTERVAL 5 MINUTES);
```

**Example input / output:**

```text
Input:
Timestamps for transactions in a stream.

Output (Tumbling Window):
2023-06-01 14:00:00 - GMV: 125.0

Output (Sliding Window):
2023-06-01 14:00:00 - GMV: 125.0
2023-06-01 14:05:00 - GMV: 175.0

Output (Session Window):
2023-06-01 14:00:00 - GMV: 125.0
2023-06-01 14:10:00 - GMV: 175.0
```

**Data:**

```json
{
  "transactions": [
    {"id": 1, "amount": 50.0, "timestamp": "2023-06-01T14:00:00"},
    {"id": 2, "amount": 75.0, "timestamp": "2023-06-01T14:01:00"},
    {"id": 3, "amount": 100.0, "timestamp": "2023-06-01T14:10:00"},
    {"id": 4, "amount": 50.0, "timestamp": "2023-06-01T14:15:00"},
    {"id": 5, "amount": 75.0, "timestamp": "2023-06-01T14:20:00"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Input Stream
8. Tumbling Window
9. Window End?
10. Aggregate GMV
11. Sliding Window
12. Is New Data?
13. Update GMV
14. Session Window
15. No New Data?
16. End Session, Aggregate GMV

