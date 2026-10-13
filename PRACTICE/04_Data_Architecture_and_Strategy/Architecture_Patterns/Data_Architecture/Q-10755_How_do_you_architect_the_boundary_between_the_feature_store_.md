---
type: "practice"
difficulty: "Expert"
---

# How do you architect the boundary between the feature store's online serving layer and a purpose-built OLAP store (Pinot/Druid) when some ML features are also real-time analytics aggregates? When do you refuse to run both?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When architecting the boundary between a feature store's online serving layer and an OLAP store like Pinot or Druid, we need to ensure that real-time analytics aggregates can seamlessly flow from the feature store while maintaining data consistency and performance. This involves designing a robust architecture that balances read-heavy operations of the OLAP store with the write-heavy nature of the feature store.

#### Triaging the Question

The question is essentially about integrating a feature store's online serving layer (which primarily handles real-time, low-latency reads) with an OLAP store like Pinot or Druid, which focuses on complex queries and analytics. The challenge lies in handling real-time analytics aggregates that require both freshness and performance.

#### Assumptions
Latency Requirements: Real-time analytics should not have a high tolerance for latency.
Data Consistency: We need to maintain data consistency between the feature store and the OLAP store.
Scalability: The solution must be scalable to handle large volumes of data.

#### First-Principles Solution

Data Ingestion:

Feature Store Online Serving Layer: This layer serves features for models in real-time, making frequent low-latency updates possible.
Data Synchronization Mechanism: We need a mechanism (like Apache Kafka or AWS Kinesis) to push updated feature values from the online serving layer to the OLAP store.

Real-Time Analytics Aggregates:

Stream Processing Engine: Use a stream processing engine like Apache Flink, Apache Spark Streaming, or AWS Kinesis Data Streams to process incoming data and generate real-time analytics aggregates.
Aggregation Strategy: Real-time aggregations can be computed on the fly based on user actions or batch processes.

OLAP Store Integration:

Pinot/Druid Schema Design: Ensure that the schema in Pinot or Druid is optimized for both read and write operations. Pinot, for instance, supports dynamic ingestion where data updates are appended to existing segments.
ETL Pipeline: Set up an ETL pipeline using tools like Apache NiFi or Apache Airflow to automate the process of moving aggregated data from the feature store to the OLAP store.

Performance Optimization:

Data Partitioning: Use data partitioning strategies (e.g., time-based, spatial-based) in Pinot/Druid for better query performance.
Indexing Strategies: Implement indexing techniques to improve query response times and reduce storage costs.

#### Closing Summary

In summary, architecting the boundary between a feature store's online serving layer and an OLAP store involves careful design of data synchronization mechanisms, real-time processing pipelines, and optimized schema designs. This ensures that both low-latency reads and complex queries are handled efficiently without compromising on data consistency or performance.

#### Common Mistakes/Follow-Ups
Overcomplicating the Synchronization Mechanism: Not choosing the right tool for stream processing can lead to inefficiencies.
Ignoring Data Consistency: Failing to ensure that changes in the feature store are reflected in the OLAP store promptly can result in inaccurate analytics.
Performance Bottlenecks: Inadequate indexing and partitioning strategies can degrade query performance over time.

**Comparison:**

```
Feature	Description
Kafka	Used for real-time data ingestion
Flink	Stream processing engine
Pinot/Druid	OLAP store
```

**Code:**

```python
# Example of setting up a Kafka topic for real-time data ingestion
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['localhost:9092'])
message = b'{"feature_key": "user_id", "feature_value": 10}'
future = producer.send('realtime_features', message)
```

**Example input / output:**

```text
Input:
- Real-time feature updates from the online serving layer

Output:
- Aggregated data in Pinot or Druid for real-time analytics queries
```

**Data:**

```json
{
  "schema": {
    "type": "record",
    "name": "FeatureEvent",
    "fields": [
      {"name": "feature_key", "type": "string"},
      {"name": "feature_value", "type": "int"}
    ]
  }
}
```

**Flow:**

1. Feature Store Online ServingLayer
2. Apache Kafka
3. Stream Processing Engine(Flink)
4. Aggregated Data in OLAPStore (Pinot/Druid)
5. Real-time Analytics Queries

