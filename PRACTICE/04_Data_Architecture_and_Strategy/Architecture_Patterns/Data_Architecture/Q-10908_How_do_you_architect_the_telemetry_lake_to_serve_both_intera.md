---
type: "practice"
difficulty: "Expert"
---

# How do you architect the telemetry lake to serve both interactive incident querying (sub-second) and heavy anomaly-model training on the same underlying data without either workload degrading the other? What materializations are required?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Reproducible Telemetry State](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Reproducible_Telemetry_State.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a telemetry lake that serves both interactive incident querying and heavy anomaly-model training, we need to balance real-time performance with historical analysis capabilities. The key is to leverage a hybrid data architecture approach using a combination of a stream processing system (like Apache Kafka or AWS Kinesis) for near-real-time data ingestion and processing, and a data warehousing solution (like Amazon Redshift, Google BigQuery, or Snowflake) for storing and querying large historical datasets.

The core idea is to create an "event mesh" where data flows through multiple layers: raw event collection, real-time streaming processing, and offline storage and analytics. By carefully designing materializations and using the right tools, we can ensure that interactive queries do not impact training tasks and vice versa.

**Comparison:**

```
Layer	Data Source	Purpose
Raw Event Collection	Kafka/Kinesis Topic	Collect raw telemetry events
Real-Time Processing	Apache Flink/KStreams	Process and transform data in real-time
Offline Storage	Data Warehouse	Store historical event data
Materialized Views	Data Warehouse	Serve interactive querying
Anomaly Model Training	Data Warehouse	Train models using historical data
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Raw Event Collection] --> B{Do we need real-time processing?}
    B -- Yes --> C[Apache Kafka/Kinesis Stream Processing]
    B -- No --> D[Directly to Warehouse]
    C --> E(Real-Time Processing) --> F[Materialized Views]
    C --> G(Offline Storage)
    G --> H[Data Warehousing] --> I[Anomaly Model Training]
    F --> I
```

**Example input / output:**

```
Example Workflow:
PLAINTEXT
Copy
Input:
- Raw event data streaming into Kafka/Kinesis topic.

Output:
- Real-time processing materialized views used for interactive querying.
- Offline storage and analytics materialized views for anomaly model training.
```

**Data:**

```

Schema Example for Kafka Topic:

JSON
Copy
{
    "topic": "telemetry_events",
    "fields": [
        { "name": "timestamp", "type": "timestamp" },
        { "name": "device_id", "type": "string" },
        { "name": "metric_value", "type": "float" }
    ]
}

Schema Example for Data Warehouse:

JSON
Copy
{
    "table": "telemetry_data",
    "fields": [
        { "name": "timestamp", "type": "datetime" },
        { "name": "device_id", "type": "string" },
        { "name": "metric_value", "type": "float" }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Raw Event
4. Raw Event Collection
5. Real-Time Processing?
6. Apache Flink/KStreams
7. Directly to Warehouse
8. Real-Time ProcessingMaterialized Views
9. Interactive Querying
10. Offline Storage
11. Data Warehousing
12. Anomaly Model Training
13. Directly to Offline Storage

