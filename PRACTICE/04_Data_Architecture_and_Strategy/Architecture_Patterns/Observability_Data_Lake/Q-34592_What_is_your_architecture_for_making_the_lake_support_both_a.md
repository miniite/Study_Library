---
type: "practice"
difficulty: "Deep"
---

# What is your architecture for making the lake support both a self-serve analyst persona and a low-latency automated AIOps consumer without one design compromising the other?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Curated Observability Datasets](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Curated_Observability_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need an observability data lake that can cater to both a self-serve analyst persona who requires ad-hoc analysis and querying of historical data, and a low-latency automated AIOps consumer that needs near real-time insights for proactive anomaly detection. The architecture must balance these conflicting requirements without one design impacting the other.

First, let's clarify what we are dealing with:

Self-serve Analyst Persona: This persona requires flexible access to historical data for ad-hoc analysis and querying.
Low-latency AIOps Consumer: This consumer needs near real-time data processing to detect anomalies and trigger automated actions quickly.

To design such an architecture, we'll leverage a hybrid approach combining real-time streams for the AIOps consumer and offline storage for self-serve analysis.

Step-by-Step Solution

Data Ingestion Layer:

Collect raw data from various sources using ingestion tools like Kafka or AWS Kinesis.
Use stream processing engines like Apache Flink or KStreams to process real-time streams, filtering and aggregating the data as necessary for AIOps.

Real-Time Processing Pipeline:

Implement a real-time analytics pipeline that processes incoming data in near-real time.
Store the processed data in a low-latency store such as Apache Druid or AWS Redshift Spectrum to enable rapid querying by the AIOps consumer.

Offline Storage Layer:

Use offline storage solutions like Hadoop Distributed File System (HDFS) or Amazon S3 to store raw and aggregated historical data.
Employ time-series databases like InfluxDB or TimeScale for storing high-frequency time series data, which can be queried later using SQL-like languages.

Query Processing Layer:

For self-serve analysts, use a query processing layer such as Apache Presto or Amazon Athena to allow ad-hoc querying of historical data.
Ensure this layer is optimized for complex queries and can handle large datasets efficiently.

Data Governance and Security:

Implement robust security measures to ensure that sensitive data is properly protected, especially when allowing self-serve access.
Apply data governance policies to manage metadata, lineage, and retention rules.

Monitoring and Alerting:

Set up monitoring and alerting mechanisms for both the real-time stream processing and offline query processes to ensure high availability and performance.

#### Closing Summary

This architecture ensures that we can support both the self-serve analyst persona and the low-latency AIOps consumer by decoupling their data access paths. The real-time stream processing layer handles near real-time analysis, while the offline storage layer supports ad-hoc querying of historical data. This separation prevents one design from compromising the other.

#### Common Mistakes/Follow-Ups
Common Mistake: Mixing real-time and batch processing in a single system can lead to performance degradation and increased complexity.

#### Follow-Up
Discussing how to balance costs and resource utilization, such as using different clusters for real-time vs. offline workloads or leveraging cost-effective storage solutions.

**Comparison:**

```
Metric	Frequency	Storage
cpu_usage	Real-time	Druid
memory_usage	Real-time	Druid
```

**Code:**

```python
# Example Python script for data ingestion (using KStreams)
from kafka import KafkaProducer
import json

def send_data_to_kafka(topic, data):
    producer = KafkaProducer(bootstrap_servers='localhost:9092')
    message = json.dumps(data).encode('utf-8')
    producer.send(topic, value=message)

# Example of sending a log entry to the stream
log_entry = {"timestamp": "2023-10-01T00:00:00Z", "level": "INFO", "message": "System initialized"}
send_data_to_kafka('logs', log_entry)
```

**Example input / output:**

| Input Data | Output Aggregated Metrics |
|------------|--------------------------|
| `{"timestamp": "2023-10-01T00:00:00Z", "metric": "cpu_usage", "value": 85}` | {"timeRange": ["2023-10-01T00:00:00Z", "2023-10-01T00:01:00Z"], "metrics": [{"metric": "cpu_usage", "average": 85.0}]} |

**Data:**

```json
{
  "rawData": [
    {"timestamp": "2023-10-01T00:00:00Z", "metric": "cpu_usage", "value": 85},
    {"timestamp": "2023-10-01T00:01:00Z", "metric": "memory_usage", "value": 75}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Real-Time Stream?
5. Stream Processing
6. Offline Storage
7. Real-Time Analytics
8. Historical Queries
9. AIOps Consumer
10. Self-serve Analyst

