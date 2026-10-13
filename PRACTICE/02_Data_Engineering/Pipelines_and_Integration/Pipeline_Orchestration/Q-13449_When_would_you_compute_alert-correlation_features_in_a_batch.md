---
type: "practice"
difficulty: "Deep"
---

# When would you compute alert-correlation features in a batch Spark job versus in a streaming job and what breaks if you pick wrong for a paging-critical use case?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batch vs Streaming Features](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine when to compute alert-correlation features using a batch Spark job versus a streaming one, we need to consider the nature of our data pipeline and the requirements of our system. Batch processing is ideal for tasks that involve historical data or can tolerate some latency, whereas streaming jobs are better suited for real-time analysis where quick responses are necessary.

In this scenario, we're dealing with alert-correlation features in a paging-critical use case. Here’s a step-by-step breakdown of the considerations and trade-offs:

Data Latency and Urgency: If alerts need to be correlated as soon as possible after an incident occurs, streaming jobs would be more appropriate because they can process data as it arrives, providing real-time insights.

Resource Management and Scaling: Batch processing is easier to manage and scale in terms of resource allocation, especially when dealing with large datasets that can be processed in stages. Streaming jobs require continuous monitoring and might need dynamic scaling based on incoming data volume.

Complexity and Maintenance: Stream processing introduces additional complexity due to the continuous nature of data intake and the potential for state management. Batch processes are simpler but may involve longer execution times, especially with complex transformations.

Error Handling and Recovery: Streaming jobs have built-in mechanisms for handling outages and errors more gracefully compared to batch jobs, which might require custom error-handling strategies if they fail mid-execution.

If we were to pick the wrong type of job for a paging-critical use case, it could lead to significant issues:

Batch Job Misuse: If we choose a batch job in a scenario where real-time insights are crucial, such as during an ongoing incident response, delays could result in missed critical alerts or delayed responses.
Streaming Job Misuse: Conversely, if we rely on a streaming job for historical analysis where it doesn’t make sense, the system might generate unnecessary noise and consume resources without providing actionable insights.

For example, consider an alert-correlation system that needs to provide immediate feedback during a production incident. If we choose a batch job, any delay in processing could result in missing critical signals or delayed response times, potentially leading to further issues down the line. On the other hand, if we use a streaming job for historical analysis, it might generate redundant alerts and consume unnecessary resources.

In conclusion, understanding the specific requirements of your system is key to making the right choice between batch and streaming jobs. For critical paging scenarios, real-time processing capabilities provided by streaming jobs are often preferred over batch processing due to their lower latency and more immediate actionable insights.

**Comparison:**

```
Input Topic	Messages (Timestamp, Alert Type)
alerts_topic	(2023-10-01T12:00:00Z, database_error)
	(2023-10-01T12:05:00Z, network_timeout)
	(2023-10-01T12:10:00Z, database_error)
Output File	Data (Time, Alert Type Count)
/data/correlated_alerts	(2023-10-01T12:10:00Z, 2 database_errors)
```

**Code:**

```python
# Example Airflow DAG for a streaming job using Spark Streaming
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("StreamingAlertCorrelation").getOrCreate()

# Define the data stream source (e.g., Kafka topic)
stream_df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "alerts_topic") \
    .load()

# Apply transformations to correlate alerts
correlated_alerts = stream_df.groupBy("alert_type").agg({"timestamp": "max"})

# Write the correlated data to a persistent storage (e.g., HDFS)
query = correlated_alerts.writeStream.outputMode("complete").format("parquet") \
    .option("path", "/data/correlated_alerts") \
    .start()

query.awaitTermination()
```

**Example input / output:**

```text
Input (Kafka Messages):
[
  {"timestamp": "2023-10-01T12:00:00Z", "alert_type": "database_error"},
  {"timestamp": "2023-10-01T12:05:00Z", "alert_type": "network_timeout"},
  {"timestamp": "2023-10-01T12:10:00Z", "alert_type": "database_error"}
]

Output (Parquet File):
[
  {
    "time": "2023-10-01T12:10:00Z",
    "alerts": [
      {"type": "database_error", "count": 2}
    ]
  }
]
```

**Data:**

```json
{
  "alerts_topic": {
    "partitions": 3,
    "messages": [
      {"timestamp": "2023-10-01T12:00:00Z", "alert_type": "database_error"},
      {"timestamp": "2023-10-01T12:05:00Z", "alert_type": "network_timeout"},
      {"timestamp": "2023-10-01T12:10:00Z", "alert_type": "database_error"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Alert Correlation System
4. Batch Processing?
5. Historical Data Analysis
6. Real-time Alert Correlation
7. Generate Reports/Insights
8. Generate Real-time Alerts

