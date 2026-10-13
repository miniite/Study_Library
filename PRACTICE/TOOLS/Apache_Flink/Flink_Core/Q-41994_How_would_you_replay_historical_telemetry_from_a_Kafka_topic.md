---
type: "practice"
difficulty: "Intermediate"
---

# How would you replay historical telemetry from a Kafka topic through the same Flink job to test a new detection rule offline?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Flink vs Spark Streaming](<../../../../TOOLS/Apache_Flink/Flink_Core/Flink_vs_Spark_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To replay historical telemetry from a Kafka topic through an existing Apache Flink stream processing job, we need to set up a replay mechanism that reads from a snapshot of past data and simulates it in real-time or near-real-time. This approach is essential for testing new detection rules without impacting ongoing production traffic.

The core idea here is to create a separate Flink job that mimics the original streaming process but uses historical data instead. We'll need to configure our replay job to read from Kafka topics, simulate the event timestamps, and feed them into the same processing pipeline as before. This will allow us to test how the new detection rule behaves with past telemetry without disrupting current operations.

**Comparison:**

| Column Name | Type      |
|-------------|-----------|
| id          | String    |
| value       | Integer   |

**Code:**

```python
# Example Python Flink streaming code for replaying historical data
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, Kafka

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

# Define the schema of your telemetry data
t_env.connect(Kafka()
              .version('0.11')
              .topic('<REPLAY_TOPIC>')
              .start_from_earliest()
              .property('zookeeper', '<ZOOKEEPER_ADDRESS>')
              .property('bootstrap.servers', '<KAFKA_BOOTSTRAP_SERVERS>'))
           .with_format(OldCsv()
                        .field('column1', DataTypes.STRING())
                        .field('column2', DataTypes.INT()))
           .with_schema(Schema()
                        .field('id', 'string')
                        .field('value', 'int'))
           .register_table_source('source_table')

t_env.register_table_sink('<SINK_TABLE>', DataTypes.ROW(
    [DataTypes.FIELD('id', DataTypes.STRING()),
     DataTypes.FIELD('value', DataTypes.INT())]),
    'print')

table = t_env.from_path('source_table')
result = table.select(table['id'], table['value'])
t_env.to_append_stream(result, DataTypes.ROW([
    DataTypes.FIELD('id', DataTypes.STRING()),
    DataTypes.FIELD('value', DataTypes.INT())])).print()
```

**Example input / output:**

```

Example input data:

JSON
Copy
[
  {"id": "1001", "value": 50},
  {"id": "1002", "value": 75},
  {"id": "1003", "value": 60}
]

Expected output (assuming a simple print operation):

PLAINTEXT
Copy
+----------+--------+
|    id    | value |
+----------+--------+
|  1001    |   50  |
|  1002    |   75  |
|  1003    |   60  |
```

**Data:**

```json
{
  "columns": ["id", "value"],
  "schema": [
    {"name": "id", "type": "string"},
    {"name": "value", "type": "integer"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Historical Data Source
4. Is Replay Topic Configured?
5. Create Replay Flink Job
6. Read from Kafka Replay Topic
7. Simulate Real-time Processing
8. Test New Detection Rule
9. Verify Results
10. Configure Replay Topic andRestart Job

