---
type: "practice"
difficulty: "Deep"
---

# How would you architect a streaming pipeline to correlate metrics, logs, and traces for the same incident when they arrive on three separate topics at different latencies? What's your join key and buffer strategy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a real-time streaming pipeline that correlates metrics, logs, and traces for the same incident across three separate topics, we need to consider several factors including latency, data consistency, fault tolerance, and efficient buffering strategies. The key is to ensure that related events can be matched even when they arrive at different times due to varying latencies.

The architecture will involve using a combination of technologies such as Apache Kafka for messaging, Apache Flink or Spark Streaming for processing, and possibly Elasticsearch or a similar service for storage and querying the correlated data. The join key will be derived based on unique identifiers like incident IDs, timestamps, or contextual metadata that can link these disparate events.

Let’s break down the solution step-by-step:

Define the Join Key: The join key could be an incident_id if a single incident is uniquely identified by this field. If not, we might need to use other attributes such as timestamp, user_id, or even more contextual data like service_name.

Buffer Strategy for Latency Handling: Given that the latencies can vary significantly, a buffer strategy is crucial. This involves setting up delayed joins or using sliding windows in our processing framework to accommodate late-arriving events.

Stream Processing Framework: Apache Flink provides robust support for stateful stream processing and windowing, which are essential for handling the correlated data from multiple sources.

Topic Designation: Each topic will be designated with specific roles: metrics (e.g., metrics_topic), logs (e.g., logs_topic), and traces (e.g., traces_topic). These topics should have appropriate schemas to capture necessary fields.

Correlation Logic: We need a mechanism to correlate the events across these topics based on the join key. This could be implemented using Flink’s stateful processing capabilities or custom logic in our application layer.

Storage and Querying: Correlated data can then be stored in Elasticsearch for efficient querying, allowing us to retrieve insights into the incident dynamically.

Here is a high-level architectural overview:

Metrics Topic
Metric Processing
Logs Topic
Log Processing
Traces Topic
Trace Processing
Correlation Logic
Correlated Data Stored in ES

This diagram illustrates how metrics, logs, and traces are processed independently but correlated via a logic layer to form a unified view.

**Comparison:**

| Incident ID | Metric Value (Metric Topic) | Log Message (Logs Topic)                | Trace ID (Traces Topic) |
|-------------|-----------------------------|----------------------------------------|------------------------|
| INC-12345   | 90.8                        | Error in service A                      | TRACE-67890            |

**Code:**

```python
# Example Python code for configuring stateful stream processing with Flink

from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

# Define the schema for input topics
metrics_schema = Schema() \
    .field("incident_id", DataTypes.STRING()) \
    .field("metric_value", DataTypes.DOUBLE())

logs_schema = Schema() \
    .field("incident_id", DataTypes.STRING()) \
    .field("log_message", DataTypes.STRING())

traces_schema = Schema() \
    .field("incident_id", DataTypes.STRING()) \
    .field("trace_id", DataTypes.STRING())

# Read from Kafka topics
t_env.connect(FileSystem().path('metrics_topic')) \
        .with_format(OldCsv()
                     .field('incident_id', 'string')
                     .field('metric_value', 'double')) \
        .with_schema(metrics_schema) \
        .register_table_source("metrics")

t_env.connect(FileSystem().path('logs_topic')) \
        .with_format(OldCsv()
                     .field('incident_id', 'string')
                     .field('log_message', 'string')) \
        .with_schema(logs_schema) \
        .register_table_source("logs")

t_env.connect(FileSystem().path('traces_topic')) \
        .with_format(OldCsv()
                     .field('incident_id', 'string')
                     .field('trace_id', 'string')) \
        .with_schema(traces_schema) \
        .register_table_source("traces")

# Join logic
result = t_env.sql_query(
    """
    SELECT metrics.incident_id, metrics.metric_value, logs.log_message, traces.trace_id
    FROM metrics
    JOIN logs ON metrics.incident_id = logs.incident_id
    JOIN traces ON metrics.incident_id = traces.incident_id
    """)

# Write the result to Elasticsearch
result.to_append_stream("file", "csv").print()
```

**Example input / output:**

```text
Input (Example):
Metrics Topic:
{
  "incident_id": "INC-12345",
  "metric_value": 90.8
}

Logs Topic:
{
  "incident_id": "INC-12345",
  "log_message": "Error in service A"
}

Traces Topic:
{
  "incident_id": "INC-12345",
  "trace_id": "TRACE-67890"
}

Output (Expected):
{
  "incident_id": "INC-12345",
  "metric_value": 90.8,
  "log_message": "Error in service A",
  "trace_id": "TRACE-67890"
}
```

**Data:**

```json
{
  "metrics_topic": {
    "fields": [
      {"name": "incident_id", "type": "string"},
      {"name": "metric_value", "type": "double"}
    ]
  },
  "logs_topic": {
    "fields": [
      {"name": "incident_id", "type": "string"},
      {"name": "log_message", "type": "string"}
    ]
  },
  "traces_topic": {
    "fields": [
      {"name": "incident_id", "type": "string"},
      {"name": "trace_id", "type": "string"}
    ]
  }
}
```

**Flow:**

1. Metrics Topic
2. Metric Processing
3. Logs Topic
4. Log Processing
5. Traces Topic
6. Trace Processing
7. Correlation Logic
8. Correlated Data Stored in ES

