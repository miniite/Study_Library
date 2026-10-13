---
type: "practice"
difficulty: "Deep"
---

# How would you design a data structure to answer 'sum of the last N samples' and 'sum in the last T seconds' for a metrics agent, and why does one need time-bucketing over a plain ring buffer?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sliding Window Aggregates](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a data structure that can efficiently answer "sum of the last N samples" and "sum in the last T seconds", we must consider both space and time efficiency, especially as the number of metrics grows over time. The primary challenge is managing the trade-off between updating the sum quickly and retrieving historical sums without excessive overhead.

#### Triaging the Question

The question asks us to design a data structure for a metrics agent that can handle two types of queries:

Sum of the last N samples: This requires keeping track of recent values in an efficient manner.
Sum in the last T seconds: This necessitates a way to sum values within a sliding time window.

#### Assumptions
The metric values are numeric (e.g., integers or floating-point numbers).
The system needs to handle a large number of metrics and high-frequency updates.
We need efficient ways to update and query the data structure.

#### First Principles Solution
1. Ring Buffer (Plain Time-Based)

A plain ring buffer can store the most recent N samples, allowing us to calculate the sum efficiently by iterating over these elements. However, it doesn’t handle time-based queries well because we would need to maintain multiple buffers for different time windows, leading to high space complexity and management overhead.

Yes
No
Sample Value
Is Time-based Query?
Ring Buffer
Iterate Over Ring Buffer
2. Time-Bucketed Data Structure

To efficiently handle time-based queries, we need a data structure that can partition the samples into fixed time intervals (buckets). Each bucket stores the sum of values within its interval.

Bucket Size: Define a small bucket size ( B ) such that each bucket represents approximately T seconds. This allows us to aggregate and retrieve sums over different time windows without needing to store every single sample.
Tree Structure: Use a tree structure (e.g., a Trie or a segment tree) where nodes represent buckets, and leaf nodes store the sum of values within their respective intervals.
Detailed Implementation

Bucketing Function:

Define a function bucketId(timestamp) that maps each timestamp to its corresponding bucket ID.

Trie-based Data Structure:

Each node in the Trie represents a time interval (bucket).
The sum of all values within a given interval is stored at the leaf nodes.
Yes
No
Sample Value
Is Time-based Query?
Trie Node Update
Update Sum in Trie
Ring Buffer Update
Query Handling:
For "sum of the last N samples", iterate over the most recent N buckets and sum their values.
For "sum in the last T seconds", traverse from the current timestamp to the start of the window, aggregating sums from relevant buckets.

#### Closing Summary

Time-bucketing is crucial for efficiently handling time-based queries without excessive space overhead. A combination of a Trie or segment tree with ring buffers can provide both efficient updates and fast query responses.

#### Common Mistakes
and Follow-Ups
Mistake: Not considering the trade-off between space complexity and efficiency.

#### Follow-Up
Discussing the impact of different bucket sizes on performance and storage requirements.
Advanced Discussion: Exploring more complex structures like a hybrid approach combining hash maps with tree structures for faster lookups.

**Comparison:**

| Bucket ID | Timestamp Range     | Sum of Values |
|-----------|---------------------|---------------|
| 0         | [1634059200, 1634059210] | 60            |

**Code:**

```python
class MetricsAgent:
    def __init__(self, bucket_size):
        self.bucket_size = bucket_size
        self.buckets = {}

    def add_sample(self, timestamp, value):
        # Bucket ID mapping function
        bucket_id = self._bucket_id(timestamp)
        if bucket_id not in self.buckets:
            self.buckets[bucket_id] = 0
        self.buckets[bucket_id] += value

    def sum_last_n_samples(self, n):
        current_time = time.time()
        total_sum = 0
        for i in range(n - 1, -1, -1):
            bucket_id = self._bucket_id(current_time - (i * self.bucket_size))
            if bucket_id in self.buckets:
                total_sum += self.buckets[bucket_id]
        return total_sum

    def sum_last_t_seconds(self, t):
        current_time = time.time()
        start_time = current_time - t
        total_sum = 0
        for timestamp in range(int(start_time), int(current_time)):
            bucket_id = self._bucket_id(timestamp)
            if bucket_id in self.buckets:
                total_sum += self.buckets[bucket_id]
        return total_sum

    def _bucket_id(self, timestamp):
        # Simple example: divide by 10 seconds
        return int(timestamp // (self.bucket_size / 2))
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Add sample at timestamp 1634059200 with value 10 | -         |
| Add sample at timestamp 1634059205 with value 20 | -         |
| Sum last N samples (N=2)                         | 30        |
| Sum in the last T seconds (T=5)                  | 30        |

**Data:**

```json
{
  "metrics": [
    {"timestamp": 1634059200, "value": 10},
    {"timestamp": 1634059205, "value": 20},
    {"timestamp": 1634059210, "value": 30}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Add Sample at Timestamp
4. Bucket ID =_bucket_id(timestamp)
5. Update Sum in Trie/Bucket
6. Skip
7. Return Sum of Values

