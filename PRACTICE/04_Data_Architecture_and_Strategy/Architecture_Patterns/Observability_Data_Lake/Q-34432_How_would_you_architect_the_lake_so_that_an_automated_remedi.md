---
type: "practice"
difficulty: "Deep"
---

# How would you architect the lake so that an automated remediation workflow can query historical context in under a second before acting?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Hot vs Cold Telemetry](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Hot_vs_Cold_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an Observability Data Lake that enables rapid querying of historical context within one second, we need to focus on three core aspects: data collection and ingestion, storage optimization, and query performance tuning. This architecture ensures that our automated remediation workflows can quickly access the necessary historical data without significant latency.

The key idea is to leverage a combination of real-time streaming systems for current telemetry and efficient batch processing pipelines for historical context. By storing both types of data in optimized formats and using distributed computing frameworks, we can achieve sub-second query performance even with large datasets.

First, let’s break down the requirements:

Data Collection: Collecting and ingesting telemetry from various sources (e.g., logs, metrics, traces).
Storage Optimization: Efficiently storing historical context to support fast queries.
Query Performance Tuning: Ensuring that queries can run quickly even with large datasets.

We will use Apache Kafka for real-time data collection and Apache Iceberg for storage optimization. For query performance tuning, we employ a combination of caching and distributed computing frameworks like Apache Spark or Presto.

**Comparison:**

```
Service	Metric Type	Value
web-server	CPU Usage	0.75
web-server	Memory Usage	0.85
```

**Code:**

```
Example code snippet to set up Kafka producer

from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['localhost:9092'])

CODE
Copy
```python
# Example code snippet for Iceberg table creation
from databricks.sdk.runtime import Runtime

runtime = Runtime()
table = runtime.create_table("observability_data_lake", schema="...")  # Define your schema here
```

**Example input / output:**

```text
Input: GET metrics for service "web-server" in the last hour
Output: Query result with metric values
```

**Data:**

```json
{
  "schema": {
    "columns": [
      {"name": "timestamp", "type": "TIMESTAMP"},
      {"name": "service_name", "type": "STRING"},
      {"name": "metric_type", "type": "STRING"},
      {"name": "value", "type": "DOUBLE"}
    ],
    "primary_keys": ["service_name", "metric_type"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Real-time Data Collection
4. Kafka Producer
5. Is data real-time?
6. AWS Kinesis Firehose
7. Iceberg Table
8. Batch Processing Pipeline
9. Cache Layer
10. Distributed Query Execution(Presto/Spark)

