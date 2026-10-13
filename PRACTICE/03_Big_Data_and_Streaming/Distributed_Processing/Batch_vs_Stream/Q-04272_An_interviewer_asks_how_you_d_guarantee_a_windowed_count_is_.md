---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks how you'd guarantee a windowed count is final so downstream can act on it. How do watermarks and allowed lateness give you that?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Watermarks and State Management](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Watermarks_and_State_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a windowed count in batch or stream processing is final, we need to address two critical concepts: watermarks and allowed lateness. Watermarks help track the progress of data streams, while allowed lateness allows some late data to be processed after its initial deadline, ensuring that all relevant data can contribute to the final count.

Watermarks are essentially timestamps that indicate when it's safe to declare a window closed. When data arrives, we update these watermarks based on the timestamps of the incoming records. If no more data with a later timestamp is expected, we consider the window closed and finalize the count. This helps in ensuring that downstream systems receive final counts.

Allowed lateness extends this by allowing late arriving data to be processed after the initial deadline has passed. This is particularly useful for ensuring completeness in scenarios where there might be delayed or out-of-order messages.

Let's break down how these concepts work together step-by-step from first principles:

Understanding Watermarks:

A watermark is a mechanism used to ensure that only timely data is considered within a window.
For example, if we have a 5-minute sliding window and the latest timestamp seen so far is T, then the watermark is T - 5 minutes ago.

Handling Late Data:

Allowed lateness allows for some degree of delay in data processing. It sets an upper bound on how late data can be before it's discarded.
For instance, if allowed lateness is set to 10 minutes, any data arriving within the next 10 minutes after its timestamp would still be considered timely.

Ensuring Finality:

By combining watermarks and allowed lateness, we can ensure that all relevant data for a window has been processed.
Once the watermark reaches a point where no more late data is expected (based on the allowed lateness), the window can be declared closed.

#### Implementation
Example:

Suppose we are processing logs from an application with a 10-minute sliding window and allowed lateness of 5 minutes.
If the latest timestamp in our stream is T, then the watermark would start at T - 10 minutes ago. As long as no more data arrives within the next 5 minutes, we can safely declare the window closed.

Here’s a step-by-step example to illustrate:

Initial State: A log stream starts arriving with timestamps.
Watermark Initialization: The watermark is set at T - 10 (10 minutes before the latest timestamp).
Data Processing:
At time T + 2, data arrives with a timestamp of T - 3. We update our watermark to max(T - 10, T - 3) = T - 3.
At time T + 5, another log arrives at T - 4. The watermark is updated to max(T - 3, T - 4) = T - 3 again.
Late Data Arrives: At T + 12, a late data point with timestamp T - 6 arrives. Since the allowed lateness is 5 minutes, this data can still be considered timely and processed.

Once no more late data (within the allowed lateness) is expected:

The watermark would eventually reach T - 10. At this point, we can safely declare the window closed.

This ensures that downstream systems receive a final count based on all relevant data.

**Comparison:**

```
Event	Timestamp	Watermark Update	Processed?	Window Finalization
1	45	T - 40	Yes	No
2	30	T - 30	Yes	No
3	20	T - 20	Yes	No
4	15	T - 15	Yes	No
5	5	T - 5	Yes	Yes
```

**Code:**

```python
# Example Python code for setting up watermarks and allowed lateness

class WindowedCounter:
    def __init__(self, window_size=10, allowed_lateness=5):
        self.window_size = window_size
        self.allowed_lateness = allowed_lateness
        self.watermark = None
        self.counts = 0

    def update_watermark(self, latest_timestamp):
        if self.watermark is None:
            self.watermark = latest_timestamp - self.window_size
        else:
            self.watermark = max(self.watermark, latest_timestamp - self.window_size)

    def process_event(self, timestamp):
        if timestamp > (self.watermark + self.allowed_lateness):
            # Late data outside allowed lateness; discard and reset watermark
            self.update_watermark(timestamp)
        else:
            self.counts += 1

    def is_window_finalized(self):
        return self.watermark >= (self.latest_timestamp - self.window_size)

# Example usage
counter = WindowedCounter(window_size=10, allowed_lateness=5)
latest_timestamp = 60

for t in [45, 30, 20, 15, 5]:
    counter.update_watermark(latest_timestamp + t)
    counter.process_event(t)

# Check if window can be finalized
print(counter.is_window_finalized())
```

**Example input / output:**

```text
# Example input and output for the provided code

counter = WindowedCounter(window_size=10, allowed_lateness=5)
latest_timestamp = 60

events = [
    {"timestamp": 45},
    {"timestamp": 30},
    {"timestamp": 20},
    {"timestamp": 15},
    {"timestamp": 5}
]

for event in events:
    counter.update_watermark(latest_timestamp + event["timestamp"])
    counter.process_event(event["timestamp"])

print(counter.is_window_finalized())  # Output: True
```

**Data:**

```json
{
  "events": [
    {"timestamp": 45},
    {"timestamp": 30},
    {"timestamp": 20},
    {"timestamp": 15},
    {"timestamp": 5}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is latest timestamp >watermark + allowed lateness?
5. Mark window finalized
6. Update watermark
7. Process event

