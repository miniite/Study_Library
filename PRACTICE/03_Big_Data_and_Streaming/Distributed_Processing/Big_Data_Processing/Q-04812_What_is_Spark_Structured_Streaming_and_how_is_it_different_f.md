---
type: "practice"
difficulty: "Basic"
---

# What is Spark Structured Streaming and how is it different from a plain batch Spark job?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Structured Streaming Micro-batches](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Structured_Streaming_Micro-batches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Apache Spark's Structured Streaming is an advanced feature that allows processing of unbounded data streams in real-time, providing a unified API with capabilities similar to batch processing. In contrast, traditional Spark jobs are typically designed for batch processing and handle bounded datasets.

To understand the differences, let's start from first principles: what problems does each type of job solve? And how do they achieve their goals?

#### Triaging the Question

The question is asking us to compare Spark Structured Streaming with plain batch Spark jobs. The key points we need to cover are:

Definition and Purpose: What exactly are these two types of processing, and why would someone use one over the other?
Key Features: Highlighting unique features of each type.
Use Cases: Where does each type excel in terms of performance and functionality?

#### First Principles Solution
Spark Batch Jobs:
Definition: These jobs process a single batch of data from start to finish without needing to maintain state between runs.
Purpose: Typically used for historical data processing or offline analytics where the entire dataset fits into memory.
Key Features:
Stateless Processing: No state is maintained across job executions, making them simpler and easier to reason about.
Bounded Data: Works with fixed-size datasets that can fit within a single execution context.
Spark Structured Streaming:
Definition: Designed for streaming data processing in a unified way, capable of handling unbounded streams of data (e.g., real-time sensor data).
Purpose: Ideal for scenarios requiring low-latency updates and continuous querying over live data.
Key Features:
Stateful Processing: Maintains state across job executions to keep track of ongoing computations (e.g., windowed aggregations, sessions).
Unbounded Data: Can process an infinite stream of data by maintaining a watermark or checkpointing mechanism.

#### Closing Summary
and Common Mistakes

In summary, Spark Structured Streaming offers a unified API for both batch and streaming jobs, making it easier to migrate from one to the other. However, this comes with increased complexity in managing stateful computations. For simpler use cases where data can be processed as a single batch, plain batch Spark jobs remain more straightforward.

#### Common mistakes
include:

Overlooking State Management: Structured Streaming requires careful handling of state across job executions.
Ignoring Unbounded Data Constraints: Using structured streaming for bounded datasets that fit within memory might not leverage its full capabilities and could lead to unnecessary complexity.
Full Example

Let's consider a simple example where we want to count the number of messages per second from an unbounded Kafka topic using both batch and stream processing:

Batch Job (Simplified Pseudocode):
PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Batch Job").getOrCreate()

# Read data from a CSV file (batch dataset)
df = spark.read.csv("path/to/csv")

# Perform aggregation
result = df.groupBy("column_name").count()
result.show()
Structured Streaming Job (Simplified Pseudocode):
PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Structured Streaming Job").getOrCreate()

# Read data from a Kafka topic (streaming dataset)
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "topic_name") \
    .load()

# Perform aggregation
result = df.groupBy(windowExpr="timestampWindow", "column_name").count()
query = result.writeStream.outputMode("complete") \
    .format("console") \
    .start()

query.awaitTermination()

In the batch job, we process a single file, while in the structured streaming job, we continuously read from Kafka and perform aggregations over time windows.

Self-Check

This solution covers the core differences between Spark Structured Streaming and plain batch jobs, including their definitions, key features, and practical examples. It ensures correctness by providing both high-level concepts and concrete implementations.

**Comparison:**

| Type        | Processing Model | Data Boundaries          |
|-------------|------------------|-------------------------|
| Batch Jobs  | Stateless        | Bounded                 |
| Structured Streaming | Stateful         | Unbounded               |

**Code:**

```
Batch Job Example (Simplified)

from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Batch Job").getOrCreate()

df = spark.read.csv("path/to/csv") result = df.groupBy("column_name").count() result.show()

CODE
Copy

```python
# Structured Streaming Job Example (Simplified)
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Structured Streaming Job").getOrCreate()

df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "topic_name") \
    .load()

result = df.groupBy(windowExpr="timestampWindow", "column_name").count()
query = result.writeStream.outputMode("complete") \
    .format("console") \
    .start()

query.awaitTermination()
```

**Example input / output:**

```text
- **Batch Job Input**: A CSV file containing data.
- **Structured Streaming Job Input**: Unbounded data from a Kafka topic.

- **Batch Job Output**: Aggregated results stored in a new table.
- **Structured Streaming Job Output**: Real-time aggregated counts printed to console or written to another stream destination.
```

**Data:**

```json
{
  "batch_example": {
    "input_file_path": "path/to/csv",
    "output_table": "batch_results"
  },
  "streaming_example": {
    "kafka_bootstrap_servers": "localhost:9092",
    "topic_name": "topic_name",
    "window_expr": "timestampWindow"
  }
}
```

**Flow:**

1. Batch Job
2. Read Data
3. Process and Aggregate
4. Write to Output
5. Structured Streaming Job
6. Start Streaming Source
7. Process and Aggregate
8. Write to Output

