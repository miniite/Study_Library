---
type: "practice"
difficulty: "Deep"
---

# When is at-least-once acceptable versus exactly-once for a warehouse-load pipeline, and how do you make the sink idempotent instead?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once into Kafka](<../../../../TOOLS/Apache_Flink/Flink_Core/Exactly_Once_into_Kafka.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

At-least-once processing is often sufficient when the cost of duplicates is lower than the risk of data loss. Exactly-once semantics are necessary in scenarios where ensuring no duplicates and no data loss is critical, such as financial transactions or medical records. To achieve exactly-once semantics with Apache Flink's stream processing, making the sink idempotent is crucial.

To summarize quickly: at-least-once is acceptable when duplicates can be handled by downstream systems, while exactly-once requires handling state and ensuring idempotency in the sinks.

**Comparison:**

```
Key	At-Least-Once	Exactly-Once
Cost of Duplicates	Low to Moderate	High
Risk of Data Loss	Tolerable	Not Tolerable
Example Use Case	Log Aggregation, Real-Time Analytics	Financial Transactions, Healthcare
```

**Code:**

```python
# Example of Flink configuration for idempotent sink
env = StreamExecutionEnvironment.get_execution_environment()
env.enable_checkpointing(5000)  # Checkpoint every 5 seconds

# Define a custom output format to ensure idempotence
class IdempotentSink(SinkFunction):
    def open(self, parameters: Configuration):
        self.writer = Writer()  # Initialize your writer here

    def process(self, value):
        try:
            self.writer.write(value)  # Safe write operation
        except Exception as e:
            log.error(f"Failed to write {value}, error: {e}")

# Register the sink in Flink
env.add_sink(IdempotentSink(), 'sink_name')
```

**Example input / output:**

```text
Input: [1, 2, 3]
Checkpoint: State updated with [1, 2, 3]

Failure during processing of 4 -> Checkpoint [1, 2, 3] is not lost

Recovery: Resuming from checkpoint [1, 2, 3]
Output: [1, 2, 3]
```

**Data:**

```json
{
  "checkpoint_interval": 5000,
  "sink": {
    "type": "IdempotentSink",
    "parameters": [
      {"key": "writer", "value": "some_writer_instance"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Event Source
4. Is Duplicates Acceptable?
5. At-Least-Once Sink
6. Exactly-Once Sink withIdempotence
7. Write to Warehouse
8. Check State Before Write
9. Write to Warehouse if NotAlready Written

