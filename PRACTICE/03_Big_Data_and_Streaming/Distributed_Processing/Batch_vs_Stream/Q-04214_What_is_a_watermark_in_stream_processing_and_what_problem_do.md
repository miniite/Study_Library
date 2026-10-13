---
type: "practice"
difficulty: "Deep"
---

# What is a watermark in stream processing and what problem does it solve?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Watermarks and State Management](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Watermarks_and_State_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Watermarks are a critical concept in stream processing, particularly when dealing with unbounded data streams where elements can arrive out of order or with delays. A watermark is essentially a timestamp that represents the latest point in time from which we have received all possible events for a given key up to that point. It helps in managing state and ensuring that late arriving data does not interfere with processing completed data.

To understand the problem watermarks solve, let's break it down:

Unbounded Data Streams: In stream processing, data can arrive continuously without any predefined end.
Event Timestamps vs Watermark Timestamps: Events have their own timestamps (event-time), which indicate when an event occurred in reality. Watermark timestamps are used to infer a point after which no more events for a particular key will be received.
The Problem Watermarks Solve

In the context of stream processing, there is often a need to aggregate data over time or across multiple elements. For example, if you're calculating a moving average, late-arriving events can disrupt the computation because they might represent older states that should have been processed already. Without proper handling, this can lead to incorrect results.

#### Example Scenario

Imagine you are processing a stream of financial transactions where each transaction has a timestamp. You want to calculate daily balances for users. If a transaction is late and arrives after the end of the day, it should not be included in that day's balance but rather processed as part of the next day’s calculations.

How Watermarks Work
Watermark Generation: For each key (e.g., user ID), the system generates a watermark at regular intervals or based on conditions. This timestamp marks the latest point in time from which no more events will be received for that key.
Watermark Propagation: The watermark is propagated through the processing topology, ensuring that any state updates before this timestamp are complete and accurate.
Late Event Handling: Late events (events arriving after a watermark) are handled by either ignoring them or reprocessing previous states.

#### Example

#### Cons
ider a simple stream of stock price updates:

Event 1: {timestamp: 09:00, stock_id: A, price: 10}
Event 2: {timestamp: 09:05, stock_id: B, price: 11}
Event 3: {timestamp: 08:50, stock_id: A, price: 9} (late event)

If we set a watermark for key A at 09:00, the late event 3 would be ignored in the current window because it is past the watermark. The system ensures that only data up to and including the watermark's timestamp is considered.

#### Common Mistakes
Ignoring Watermarks: Failing to set or propagate watermarks can lead to incorrect state management.
Incorrect Watermark Scheduling: Not setting watermarks at appropriate intervals or conditions can result in unnecessary reprocessing of events.
Late Event Mismanagement: Handling late events incorrectly (e.g., including them in the current window) can skew results.

#### Closing Summary

Watermarks are essential for maintaining accurate and consistent state in stream processing, especially when dealing with unbounded data streams. By ensuring that only relevant data is processed up to a certain point, watermarks prevent issues like incorrect results from late-arriving events. Understanding how to correctly implement and use watermarks can significantly enhance the robustness of your stream processing applications.

**Comparison:**

```
Timestamp	Stock ID	Price
1632450000	A	10
1632450300	B	11
1632449700	A	9
Watermark for Key 'A' at 1632450000.
Late Event 3 (Timestamp: 1632449700) ignored.
```

**Code:**

```java
// Example Java code snippet showing watermark generation
public class WatermarkGenerator {
    private long lastWaterMarkTime = 0;

    public void onEvent(long timestamp) {
        if (timestamp > lastWaterMarkTime) {
            lastWaterMarkTime = timestamp;
            System.out.println("Watermark generated: " + lastWaterMarkTime);
        }
    }
}
```

**Example input / output:**

```text
Input:
- Event 1: Timestamp: 1632450000 (9:00), Stock ID: A, Price: 10
- Event 2: Timestamp: 1632450300 (9:05), Stock ID: B, Price: 11
- Late Event 3: Timestamp: 1632449700 (8:50), Stock ID: A, Price: 9

Watermark generated for Key 'A' at 1632450000.

Late Event 3 is ignored as it's older than the watermark.
```

**Data:**

```json
[
  { "timestamp": 1632450000, "stock_id": "A", "price": 10 },
  { "timestamp": 1632450300, "stock_id": "B", "price": 11 },
  { "timestamp": 1632449700, "stock_id": "A", "price": 9 }
]
```

**Flow:**

1. No
2. Yes
3. Event
4. Is Event Older ThanWatermark?
5. Ignore Late Event
6. Include In Current Window

