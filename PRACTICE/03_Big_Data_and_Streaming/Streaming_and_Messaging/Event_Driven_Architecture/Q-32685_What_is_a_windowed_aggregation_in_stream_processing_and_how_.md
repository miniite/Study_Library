---
type: "practice"
difficulty: "Intermediate"
---

# What is a windowed aggregation in stream processing, and how do tumbling and sliding windows differ?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fat vs Thin Events](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Windowed aggregation in stream processing involves grouping elements of a data stream over a specific time or size-based window to perform computations such as counting, averaging, summing, etc., on these groups. The core idea is that instead of processing each element individually and cumulatively, you process the data in chunks, which can lead to more efficient and scalable architectures.

The question delves into two specific types of windows: tumbling windows and sliding windows. Let's break down what they are and how they differ:

#### Triage
What is being asked: The interviewer is probing your understanding of different windowing techniques in stream processing.
Key assumptions: We will assume the context is real-time data streams where efficient aggregation is crucial.
Step-by-step solution:
Define tumbling and sliding windows.
Explain how they differ in terms of time intervals, overlap, and use cases.
Tumbling Windows

A tumbling window divides the stream into non-overlapping segments based on a fixed interval. For example, if you set a window size of 5 minutes, every 5-minute period will be treated as an independent group for processing. The key characteristics are:

No overlap between windows.
Each element belongs to only one specific window at any given time.
Sliding Windows

In contrast, a sliding window processes overlapping segments over the stream. For instance, if you set a sliding interval of 5 minutes with a gap of 1 minute, then each new 5-minute segment will overlap by 4 minutes with the previous one. The key characteristics are:

Overlapping windows.
Each element can potentially be part of multiple windows.
Detailed Explanation

Tumbling Windows: Imagine you have a data stream representing stock trades happening every second. If you apply a tumbling window of 5 seconds, each trade will fall into one of the non-overlapping 5-second buckets for processing. This approach ensures that no trade is counted twice and simplifies state management.

"State Machine"
"Aggregation Phase"
"Tumbling Window [0-4s]"
"Next Trade"
"State Machine"
"Aggregation Phase"
"Tumbling Window [0-4s]"
"Next Trade"
5 second interval
Trade
Process Trade
Update State
State Update

Sliding Windows: Now, if you apply a sliding window with a 4-second overlap (i.e., each new 5-second window starts after the previous one finishes), then trades at the edges will be shared between windows. This can provide more granular and up-to-date aggregation but requires careful state management.

"State Machine"
"Aggregation Phase"
"Sliding Window [0-4s]"
"Next Trade"
"State Machine"
"Aggregation Phase"
"Sliding Window [0-4s]"
"Next Trade"
4 second overlap, 5 second interval
Trade
Process Trade
Update State
State Update

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Tumbling windows are simpler but less granular, while sliding windows offer more up-to-date insights at the cost of complexity in state management.

#### Common Mistakes
Misunderstanding when to use each type based on the application requirements.
Overlooking the need for proper state handling and consistency checks.
Closing Thoughts

Understanding both tumbling and sliding windows is essential for designing efficient stream processing systems. The choice between them depends on whether you prioritize simplicity and fixed intervals (tumbling) or real-time updates and overlapping intervals (sliding).

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Window Types Comparison
    dateFormat YYYY-MM-DD
    section Tumbling Windows
    Tumbling Non-Overlap: 2023-10-01 - 2023-10-05, completed
    section Sliding Windows
    Sliding With Overlap: 2023-10-01 - 2023-10-05, active
```

**Code:**

```python
# Example code snippet to demonstrate windowing in a hypothetical stream processor
class TumblingWindowAggregator:
    def __init__(self, interval):
        self.interval = interval

    def process_trade(self, trade):
        # Implement logic for processing trades in non-overlapping intervals
        pass

class SlidingWindowAggregator:
    def __init__(self, interval, overlap):
        self.interval = interval
        self.overlap = overlap

    def process_trade(self, trade):
        # Implement logic for processing trades with overlapping intervals
        pass
```

**Example input / output:**

| Input (Tumbling Window) | Output |
|-------------------------|--------|
| Trade at `2023-10-01T09:00:07` in `[6s, 11s]` window | Aggregated value for `[6s, 11s]` |
| Input (Sliding Window) | Output |
|------------------------|--------|
| Trade at `2023-10-01T09:00:07` in `[6s, 11s]` window with overlap `4s` | Aggregated value for `[5s, 10s]`, `[6s, 11s]`, and `[7s, 12s]` windows |

**Data:**

```json
{
  "trades": [
    { "timestamp": "2023-10-01T09:00:05Z", "price": 100 },
    { "timestamp": "2023-10-01T09:00:10Z", "price": 105 },
    { "timestamp": "2023-10-01T09:00:15Z", "price": 110 }
  ]
}
```

**Flow:**

1. Trade Occurs
2. Process Trade
3. Update State
4. State Update
5. TradeSystem
6. Window1
7. Aggregator1
8. StateMachine1

