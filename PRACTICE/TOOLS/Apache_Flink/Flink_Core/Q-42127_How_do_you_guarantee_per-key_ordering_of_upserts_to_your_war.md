---
type: "practice"
difficulty: "Deep"
---

# How do you guarantee per-key ordering of upserts to your warehouse when Flink parallelizes processing across subtasks?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once into Kafka](<../../../../TOOLS/Apache_Flink/Flink_Core/Exactly_Once_into_Kafka.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure per-key ordering of upserts in Apache Flink, we need a robust understanding of how Flink processes data and how upsert operations are handled within a distributed system. This involves leveraging Flink's state management mechanisms effectively to maintain the required order. Let’s break down the solution step by step.

The question is essentially asking for a method to guarantee that updates (upserts) to records in your warehouse, based on keys, are processed in the correct sequence, even as Flink parallelizes the processing across multiple subtasks. This involves ensuring that each key sees its latest update before any other operations, and that these operations are applied consistently across all subtasks.

We will assume a typical scenario where you have a streaming source feeding data into Flink, which then processes this data to upsert records in a warehouse. We need to ensure that the order of updates is preserved, meaning if multiple records with the same key arrive, they should be processed one at a time before any subsequent updates.

Step-by-Step Solution

Identify Key Characteristics:

Flink processes data in parallel across subtasks.
Per-key ordering is crucial to maintain consistency when upserting into the warehouse.

Understand State Management:

Flink uses stateful processing, where each subtask maintains its own state.
For per-key ordering, we need to ensure that all updates for a particular key are processed in sequence before any other updates.

State Backend Configuration:

Use an appropriate state backend like RocksDB or HDFS to store the state data persistently.
Configure Flink’s checkpointing mechanism to periodically snapshot the state to ensure fault tolerance and recovery.

Keyed State:

Utilize keyed state in Flink, which groups elements by key. This ensures that all elements with the same key are processed in a single subtask.
By default, Flink guarantees exactly-once processing semantics within a subtask for keyed streams using operator chaining and windowing.

Upsert Logic:

Implement upsert logic where each update is treated as an insert if no existing record exists or an update if there is already a record with the same key.
Ensure that your Flink job properly handles the ProcessFunction API, which allows you to process elements and maintain state.

Stateful Processing:

Use Flink's Keyed State to store the latest version of each key in memory or persistently.
When an update comes in, check if a record with the same key exists. If it does, perform an upsert; otherwise, add a new entry.

Testing and Validation:

Test your solution thoroughly using various edge cases (e.g., late events, out-of-order data) to ensure robustness.
Validate that the state is correctly maintained across subtasks and that no records are lost or duplicated.

#### Closing Summary

Guaranteeing per-key ordering of upserts in a Flink environment requires leveraging keyed state management and proper stateful processing. By using operators like Keyed State and implementing appropriate upsert logic, you can ensure that updates to the warehouse are processed correctly and consistently across parallel subtasks.

#### Common Mistakes
and Follow-Ups
Forgetting Checkpointing: Failing to enable checkpointing can lead to data loss during failures.
Incorrect Key Assignment: Poor key assignment can result in uneven distribution of keys, leading to imbalanced processing.
State Management Issues: Incorrectly managing state can cause stale or inconsistent updates.

Would you like me to elaborate on any specific aspect, such as the implementation details or further optimizations?

**Comparison:**

| id | value |
|----|-------|
| 1  | A     |
| 2  | B     |
| 3  | C     |

**Code:**

```python
# Example Flink streaming job with upsert logic using Keyed State

from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem
from pyflink.common.typeinfo import Types
from pyflink.java_gateway import get_gateway

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

# Define schema and input source
t_env.connect(FileSystem().path('input'))
     .with_format(OldCsv()
                  .field('id', DataTypes.INT())
                  .field('value', DataTypes.STRING()))
     .with_schema(Schema()
                  .field('id', DataTypes.INT())
                  .field('value', DataTypes.STRING()))
     .register_table_source("source")

t_env.connect(FileSystem().path('output'))
     .with_format(OldCsv()
                  .field('id', DataTypes.INT())
                  .field('value', DataTypes.STRING()))
     .with_schema(Schema()
                  .field('id', DataTypes.INT())
                  .field('value', DataTypes.STRING()))
     .register_table_sink("sink")

# Define upsert logic
t_env.from_path("source") \
    .group_by("id") \
    .select(
        "id, value AS new_value",
        "old_value := state.value as old_value",  # Initialize state with None
        "if(isnotnull(old_value), if(value > old_value, value, old_value), value)"
    ) \
    .insert_into("sink")

t_env.execute("Upsert Example")
```

**Example input / output:**

```

Input:

JSON
Copy
[{"id": 1, "value": "A"}, {"id": 2, "value": "B"}, {"id": 3, "value": "C"}]

Output:

JSON
Copy
[{"id": 1, "value": "A"}, {"id": 2, "value": "B"}, {"id": 3, "value": "C"}]
```

**Data:**

```json
{
  "id": [1, 2, 3, 4],
  "value": ["A", "B", "C", "D"]
}
```

**Flow:**

1. Input Stream
2. Keyed State Initialization
3. Receive Event
4. Update Keyed State
5. Generate Output
6. Output Stream

