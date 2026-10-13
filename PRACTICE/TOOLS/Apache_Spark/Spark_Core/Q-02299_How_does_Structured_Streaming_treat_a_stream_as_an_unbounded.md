---
type: "practice"
difficulty: "Deep"
---

# How does Structured Streaming treat a stream as an unbounded table?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Shuffle Tuning and Broadcast Joins](<../../../../TOOLS/Apache_Spark/Spark_Core/Shuffle_Tuning_and_Broadcast_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Structured Streaming in Apache Spark treats an unbounded data stream as if it were an unending, but consumable, table. This approach allows us to write SQL-like queries on streams just as we would on static tables, making processing dynamic and scalable while maintaining the benefits of structured data.

To understand how Structured Streaming handles this, let's break down the core concepts and steps involved.

**Comparison:**

```
Timestamp	Value
2023-10-01T08:00:00Z	10
2023-10-01T08:01:00Z	15
2023-10-01T08:02:00Z	20
```

**Code:**

```python
# Example code snippet for creating a streaming DataFrame/Table in Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("StructuredStreamingExample").getOrCreate()
streaming_df = spark.readStream.format("kafka") \
    .option("bootstrap.servers", "localhost:9092") \
    .option("subscribe", "topic_name") \
    .load()

# Convert the DataFrame to a streaming Table
streaming_table = streaming_df.writeStream.outputMode("append").start()
```

**Example input / output:**

```
Example input data stream and the resulting output table after processing

streaming_df = spark.createDataFrame( [ (datetime.datetime(2023, 10, 1, 8, 0), 10), (datetime.datetime(2023, 10, 1, 8, 1), 15), (datetime.datetime(2023, 10, 1, 8, 2), 20) ], ["timestamp", "value"] )

Output after applying a simple query like calculating the average value

result = streaming_df.groupBy(F.avg("value")).collect() print(result)

CODE
Copy
Output:
```plaintext
[Row(avg(value)=15.0)]
```

**Data:**

```json
{
  "data": [
    {"timestamp": "2023-10-01T08:00:00Z", "value": 10},
    {"timestamp": "2023-10-01T08:01:00Z", "value": 15},
    {"timestamp": "2023-10-01T08:02:00Z", "value": 20}
  ]
}
```

**Flow:**

1. Data Source
2. Stream Processing
3. Structured Streaming Engine
4. Streaming DataFrame/Table
5. SQL-like Query Processing
6. Result Output

