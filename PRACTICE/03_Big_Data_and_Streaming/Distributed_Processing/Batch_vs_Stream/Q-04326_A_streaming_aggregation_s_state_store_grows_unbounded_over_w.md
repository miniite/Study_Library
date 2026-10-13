---
type: "practice"
difficulty: "Deep"
---

# A streaming aggregation's state store grows unbounded over weeks. How do you decide the right watermark and state TTL without dropping legitimate late events?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Watermarks and State Management](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Watermarks_and_State_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of an unbounded state store in a streaming aggregation, we need to carefully balance between minimizing state retention time (to manage storage) and ensuring that legitimate late events are not dropped. The core idea is to set appropriate watermarking strategies and state TTLs that respect the temporal constraints of the data while adhering to business requirements.

In this scenario, we’re dealing with streaming aggregation where data streams can arrive at any point in time, but there’s a practical limit to how far back late events should be considered relevant. This problem often arises in systems processing financial transactions or real-time analytics applications that require timely updates but also need to avoid holding on to stale state indefinitely.

#### Triaging the Question

The question is essentially about finding the right watermark and state TTL settings for a streaming system without dropping legitimate late events. The interviewer is probing your understanding of:

Watermarking: How to define a watermark that indicates when data should be considered "late" or outdated.
State TTL (Time-To-Live): How long to retain the state before considering it obsolete.
Late Event Handling: Ensuring that relevant late events are processed correctly.

#### First-Principles Solution

Understand the Data Latency:

Determine the maximum acceptable latency for your application. For example, in a financial trading system, transactions might need to be re-evaluated up to 24 hours after they occur.

Define Watermarks:

A watermark is essentially a timestamp that marks when data older than this value should no longer be considered valid. If an event arrives late and its timestamp is older than the current watermark, it will be discarded or marked as invalid.

Set State TTL:

The state TTL defines how long to retain information before it’s considered stale. For instance, if you determine that transactions can be re-evaluated up to 24 hours after their occurrence, setting a state TTL of 24 hours ensures the system retains relevant data for this period.

Implement Late Event Handling:

Design your system to handle late events gracefully. This could involve buffering them temporarily or marking them as special cases that need additional processing.

Monitor and Optimize:

Continuously monitor the performance of your streaming pipeline to ensure that legitimate late events are not being dropped. Adjust watermarks and state TTLs based on observed patterns and business needs.

#### Closing Summary
and Common Mistakes

The key is to strike a balance between retaining necessary data for processing and managing storage costs efficiently. A common mistake is setting overly aggressive watermarks or state TTLs, which can lead to either excessive storage usage or loss of relevant late events. It’s crucial to have clear business rules and monitor the system closely.

Common Follow-Ups
How do you determine the maximum acceptable latency?
What are the trade-offs between storage costs and data freshness in your design?
Can you walk through a scenario where an event arrives as a late event, and how it is handled?

**Comparison:**

| Event Time       | Value |
|------------------|-------|
| 2023-10-01T09:00:00Z | 1     |
| 2023-10-01T09:15:00Z | 2     |

**Code:**

```python
# Example Python code for setting watermarks and state TTL using Apache Flink
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.window import Tumble

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

# Define a watermark with a delay of 24 hours
watermark_strategy = Tumble.over(Duration.of_hours(1)).lag(Duration.of_hours(24))

# Example table schema
schema = DataTypes.ROW([
    ('event_time', DataTypes.TIMESTAMP()),
    ('value', DataTypes.INT())
])

# Register the input table with a watermark strategy
t_env.register_table_source('input', 'streaming_data', schema)
t_env.register_table_sink(
    'output',
    "select * from input",
    schema,
    watermark_strategy=watermark_strategy
)

# State TTL example
t_env.create_temporary_view("agg_view", """
    SELECT event_time, COUNT(value) as count
    FROM input
    GROUP BY TUMBLE(event_time, INTERVAL '1' MINUTE), value
""")
```

**Example input / output:**

```text
Input (example events):
2023-10-01T09:00:00Z, 1
2023-10-01T09:15:00Z, 2

Watermarked and aggregated output:
2023-10-01T09:00:00Z - 3 (sum of events in the first minute)

Late event handling:
2023-10-01T10:16:00Z, 3 (processed as a late event)
```

**Data:**

```json
{
  "events": [
    {"event_time": "2023-10-01T09:00:00Z", "value": 1},
    {"event_time": "2023-10-01T09:15:00Z", "value": 2},
    {"event_time": "2023-10-01T10:16:00Z", "value": 3}
  ]
}
```

**Flow:**

1. Input Event
2. Is event late?
3. Discard late event
4. Mark as late and processseparately
5. Output aggregated statewithout the late event
6. Buffer or delay processing oflate event
7. Output final result withprocessed data

