---
type: "practice"
difficulty: "Deep"
---

# An online feature aggregation needs both 'sum of last N events' and 'sum over last T seconds' per entity; why does the time-window version require bucketing over a plain ring buffer, and how do you bound bucket count?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sliding Window Aggregates](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address an online feature aggregation requirement that needs both "sum of last N events" and "sum over last T seconds" per entity, we need to manage data efficiently while supporting multiple time windows. The key challenge is how to handle the "sum over last T seconds" in a way that balances efficiency and performance.

#### Triaging the Question

The question asks why the time-window version requires bucketing over a plain ring buffer, and how to bound the bucket count. This involves understanding how different data structures can manage time windows effectively, specifically focusing on the trade-offs between memory usage and query performance.

#### First Principles Solution
Ring Buffer vs. Buckets for Time Windows

A ring buffer is a fixed-size circular data structure that overwrites old data as new data is added. For "sum of last N events," this works well because we only need to keep track of the most recent N items. However, managing time windows poses additional complexity.

Why Buckets Are Needed

To support "sum over last T seconds" efficiently, we cannot rely on a simple ring buffer alone. A bucketing approach involves dividing the data into fixed-size intervals (buckets) based on timestamps. Each bucket contains data points that fall within its time range. This allows us to aggregate data quickly by summing up values within specific buckets.

Bounding Bucket Count

Bounding the number of buckets is crucial for performance and memory management. The goal is to balance between precision in time window handling and resource usage.

Bucket Size: Choose a bucket size that aligns with T seconds. For example, if T = 60 seconds, each bucket could represent one minute.
Number of Buckets: To cover the required time period (e.g., an hour), we calculate the number of buckets needed: ( \text{number_of_buckets} = \frac{\text{total_time}}{T} ).
Detailed Solution

To implement this, follow these steps:

Define Time Buckets: Create a list or array where each element represents a bucket covering T seconds.
Insert New Events: When a new event arrives, insert it into the appropriate time bucket based on its timestamp.
Aggregation Logic:
For "sum of last N events," simply keep track of the latest N events.
For "sum over last T seconds," sum up all values in the relevant buckets.

#### Example

#### Cons
ider an event stream where each event has a timestamp and value. We want to maintain both metrics for each entity:

Entities: User IDs (e.g., 1, 2)
Events: (timestamp, user_id, value)

For T = 60 seconds:

PYTHON
buckets = [Bucket(), Bucket()]  # Each bucket holds values within 60s windows

def handle_event(timestamp, user_id, value):
    for bucket in buckets:
        if timestamp >= bucket.start_time and timestamp < (bucket.start_time + T):
            bucket.add_value(user_id, value)

#### Closing Summary

By using a bucketing approach over a ring buffer, we can efficiently manage time windows for both "sum of last N events" and "sum over last T seconds." The key is to define appropriate buckets based on the required time window T and bound the number of buckets to ensure optimal performance.

#### Common Mistakes/Follow-Ups
Ignoring Memory Constraints: Failing to bound the bucket count can lead to excessive memory usage.
Incorrect Bucket Sizing: Choosing an inappropriate bucket size (too large or too small) can affect query performance and accuracy.
Overlooking Edge Cases: Not handling edge cases like overlapping events or time drift.

**Comparison:**

| Timestamp | Bucket Start Time | Bucket End Time |
|-----------|------------------|----------------|
| 1634520000| 1634520000        | 1634520059     |
| 1634520010| 1634520000        | 1634520059     |
| 1634520020| 1634520000        | 1634520059     |
| 1634520030| 1634520000        | 1634520059     |

**Code:**

```python
class Bucket:
    def __init__(self, start_time):
        self.start_time = start_time
        self.values = {}

    def add_value(self, user_id, value):
        if user_id in self.values:
            self.values[user_id] += value
        else:
            self.values[user_id] = value

def handle_event(timestamp, user_id, value):
    T = 60  # Time window in seconds
    buckets = [Bucket(t) for t in range(timestamp - (timestamp % T), timestamp - (timestamp % T) + T, T)]

    for bucket in buckets:
        if bucket.start_time <= timestamp < (bucket.start_time + T):
            bucket.add_value(user_id, value)
```

**Example input / output:**

```text
# Example input and output for handling events

events = [
    {"timestamp": 1634520000, "user_id": 1, "value": 10},
    {"timestamp": 1634520010, "user_id": 1, "value": 5},
    {"timestamp": 1634520020, "user_id": 1, "value": 7},
    {"timestamp": 1634520030, "user_id": 1, "value": 8}
]

T = 60
buckets = [Bucket(t) for t in range(events[0]['timestamp'] - (events[0]['timestamp'] % T), events[0]['timestamp'] - (events[0]['timestamp'] % T) + T, T)]

for event in events:
    handle_event(event['timestamp'], event['user_id'], event['value'])

# Example output for buckets
buckets = [
  {start_time: 1634520000, values: {1: 20}},
  {start_time: 1634520010, values: {1: 5}},
  {start_time: 1634520020, values: {1: 7}},
  {start_time: 1634520030, values: {1: 8}}
]
```

**Data:**

```json
[
  {"timestamp": 1634520000, "user_id": 1, "value": 10},
  {"timestamp": 1634520010, "user_id": 1, "value": 5},
  {"timestamp": 1634520020, "user_id": 1, "value": 7},
  {"timestamp": 1634520030, "user_id": 1, "value": 8}
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Event arrives
6. Is event timestamp >= currentbucket start?
7. Add value to bucket
8. Create new bucket
9. Increment bucket index
10. Update sum of values
11. Check if bucket exceeds Tseconds
12. Reset bucket and update sum
13. Bucket handling complete

