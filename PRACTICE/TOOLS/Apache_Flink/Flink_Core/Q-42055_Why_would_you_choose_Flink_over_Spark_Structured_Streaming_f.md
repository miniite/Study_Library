---
type: "practice"
difficulty: "Deep"
---

# Why would you choose Flink over Spark Structured Streaming for a low-latency payments enrichment pipeline, and where does the choice stop mattering?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Flink vs Spark Streaming](<../../../../TOOLS/Apache_Flink/Flink_Core/Flink_vs_Spark_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between Apache Flink and Spark Structured Streaming (SSS) for a low-latency payments enrichment pipeline, several factors come into play, such as performance requirements, fault tolerance, state management, ease of development, and cost. Flink generally outperforms SSS in terms of end-to-end latency due to its event-time processing model and windowing capabilities. However, the choice can stop mattering when the additional features or constraints of SSS no longer pose significant disadvantages.

Flink is chosen for its strong support for low-latency processing with its stateful stream processing model, which allows for complex state handling at scale without the need to constantly write to external storage like Kinesis in SSS. Flink's checkpointing mechanism ensures robust fault tolerance, and its ability to process data in a time-sorted manner can significantly reduce latency.

However, if the pipeline is simple and does not require intricate state management or complex window operations, and if cost and ease of development are more critical factors, then Spark Structured Streaming might be sufficient. SSS simplifies some development tasks due to its integration with Spark’s DataFrame API, making it easier to work with structured data.

**Comparison:**

| Input Data                  | Output Data                |
|-----------------------------|----------------------------|
| field1: payment_1, field2: 100, timestamp_column: 2023-10-01T15:00:00Z | field1: payment_1, end_time: 2023-10-01T15:00:00Z, total_field2: 100 |
| field1: payment_2, field2: 200, timestamp_column: 2023-10-01T16:00:00Z | field1: payment_2, end_time: 2023-10-01T16:00:00Z, total_field2: 200 |

**Code:**

```python
# Example Flink Job for Payments Enrichment
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

t_env.connect(FileSystem().path('input')) \
    .with_format(OldCsv() \
        .field('field1', 'string') \
        .field('field2', 'long') \
        .field('field3', 'double')) \
    .with_schema(Schema() \
        .field('field1', DataTypes.STRING()) \
        .field('field2', DataTypes.BIGINT()) \
        .field('field3', DataTypes.DOUBLE())) \
    .register_source('input')

t_env.connect(FileSystem().path('output')) \
    .with_format(OldCsv() \
        .field('field1', 'string') \
        .field('field4', 'long')) \
    .with_schema(Schema() \
        .field('field1', DataTypes.STRING()) \
        .field('field4', DataTypes.BIGINT())) \
    .register_sink('output')

t_env.scan('input') \
    .assign_time_attributes('timestamp_column', 'timestamp_attribute') \
    .window(Tumble.over('1.hour').on('timestamp_column').as('w')) \
    .group_by('field1, w') \
    .select('field1, w.end as end_time, sum(field2) as total_field2') \
    .insert_into('output')

t_env.execute("Flink Payments Enrichment Pipeline")
```

**Example input / output:**

```text
# Example input data
input_data = [
    {"field1": "payment_1", "field2": 100, "timestamp_column": "2023-10-01T15:00:00Z"},
    {"field1": "payment_2", "field2": 200, "timestamp_column": "2023-10-01T16:00:00Z"}
]

# Example output data
output_data = [
    {"field1": "payment_1", "end_time": "2023-10-01T15:00:00Z", "total_field2": 100},
    {"field1": "payment_2", "end_time": "2023-10-01T16:00:00Z", "total_field2": 200}
]
```

**Data:**

```json
{
  "input": [
    {"field1": "payment_1", "field2": 100, "timestamp_column": "2023-10-01T15:00:00Z"},
    {"field1": "payment_2", "field2": 200, "timestamp_column": "2023-10-01T16:00:00Z"}
  ],
  "output": [
    {"field1": "payment_1", "end_time": "2023-10-01T15:00:00Z", "total_field2": 100},
    {"field1": "payment_2", "end_time": "2023-10-01T16:00:00Z", "total_field2": 200}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Input Data Source
4. Is the Pipeline Complex?
5. Flink's Stateful Processing
6. Spark Structured Streaming
7. Output to External Storage
8. DataFrame API Integration

