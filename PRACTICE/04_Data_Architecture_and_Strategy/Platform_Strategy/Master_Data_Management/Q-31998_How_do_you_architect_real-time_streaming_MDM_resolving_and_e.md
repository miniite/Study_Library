---
type: "practice"
difficulty: "Expert"
---

# How do you architect real-time / streaming MDM — resolving and enriching an entity within an event pipeline at sub-second latency — and where does batch reconciliation still have to backstop it?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ MDM for Batch and Streaming](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/MDM_for_Batch_and_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Master Data Management (MDM) in a real-time streaming context involves ensuring that entities are consistently resolved and enriched with up-to-date information at extremely low latency, often sub-second. This task requires a robust architecture that can handle the high volume, velocity, and variety of data while maintaining data integrity and consistency. The solution must balance the speed requirements of real-time processing with the need for periodic batch reconciliation to ensure historical accuracy.

To architect this system effectively, we start by understanding the problem: entities in our streaming pipeline need to be identified, resolved, and enriched based on incoming events, with a focus on sub-second latency. Batch reconciliation is still necessary as it backs up the real-time system by ensuring long-term data integrity and consistency through periodic processing of historical data.

#### Triage

The question probes your ability to design a scalable and robust MDM architecture that can handle streaming data efficiently while maintaining high-quality data integrity. It also tests your understanding of when batch processing is necessary as an additional safeguard against potential real-time system failures or data inconsistencies.

#### Assumptions
The entity resolution process should be done in near-real time.
There are multiple sources feeding into the pipeline, including both structured and unstructured data.
Historical accuracy and consistency must be maintained through periodic reconciliation.
Latency requirements for resolving and enriching entities are sub-second.

#### First Principles Solution

Event Pipeline Architecture

Input Streams: Real-time data from various sources (e.g., IoT devices, web logs).
Message Broker: Kafka or Apache Pulsar to manage high-volume real-time streams.
Processing Layer: Stream processing framework like Apache Flink or Spark Streaming for near-real-time processing.

Entity Resolution and Enrichment

Event Sourcing: Log all events related to entity changes (e.g., updates, deletions).
Caching: Use in-memory caching systems like Redis for quick lookup of entities.
Deduplication: Implement deduplication logic to ensure unique entity identification within the stream.

Real-Time Processing

Join Operations: Perform joins with master data stores (e.g., relational databases, NoSQL) to enrich entities in real-time.
Temporal Data Handling: Use temporal data structures to handle historical and current states of entities accurately.

Batch Reconciliation Workflow

Historical Data Processing: Periodically run batch jobs using frameworks like Apache Airflow or Kubernetes to process historical data.

#### Cons
istency Checks: Perform consistency checks between real-time and batch systems to ensure integrity.
Conflict Resolution: Implement conflict resolution strategies (e.g., majority voting, manual review) for inconsistencies detected.

Monitoring and Alerting

Latency Metrics: Monitor processing latency and alert on any deviations from the sub-second target.
Error Handling: Implement robust error handling to recover from temporary failures without disrupting service.

#### Closing Summary

To architect a real-time MDM system that resolves and enriches entities at sub-second latency, we leverage streaming technologies for near-real-time data processing. However, batch reconciliation is still necessary as it ensures long-term consistency by processing historical data. This hybrid approach balances the speed requirements of real-time systems with the need for periodic integrity checks.

#### Common Mistakes/Follow-Ups
Overreliance on Real-Time Systems: Failing to properly backstop real-time systems with batch reconciliation can lead to data inconsistencies.
Ignoring Historical Data Integrity: Underestimating the importance of historical data accuracy and failing to perform periodic consistency checks.
Inadequate Error Handling: Overlooking the implementation of robust error handling mechanisms can result in system failures without recovery.

**Comparison:**

```
Entity ID	Name	Address
1001	John Doe	123 Elm St, NYC
1002	Jane Smith	456 Oak St, LA
```

**Code:**

```python
# Example Python code snippet for real-time processing using Flink
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

t_env.connect(FileSystem().path('input_path')) \
    .with_format(OldCsv()
                 .field('id', DataTypes.STRING())
                 .field('value', DataTypes.INT())) \
    .with_schema(Schema()
                 .field('id', DataTypes.STRING())
                 .field('value', DataTypes.INT())) \
    .register_table_source('mySource')

t_env.connect(FileSystem().path('output_path')) \
    .with_format(OldCsv()
                 .field('id', DataTypes.STRING())
                 .field('enriched_value', DataTypes.INT())) \
    .with_schema(Schema()
                 .field('id', DataTypes.STRING())
                 .field('enriched_value', DataTypes.INT())) \
    .register_table_sink('mySink')

t_env.from_path('mySource') \
    .join(t_env.from_path('master_data_store')) \
    .select(['id', 'value', 'enriched_value']) \
    .to_append_stream(t_env, FileSystem().path('output_path'))
```

**Example input / output:**

```

Input Event:

JSON
Copy
{
  "id": "123",
  "value": "example_value"
}

Output Enriched Event (after real-time processing):

JSON
Copy
{
  "id": "123",
  "enriched_value": "enhanced_example_value"
}
```

**Data:**

```json
{
  "event": {
    "id": "123",
    "timestamp": "2023-10-05T14:00:00Z",
    "value": "example_value"
  }
}
```

**Flow:**

1. Real-time Event Stream
2. Event Sourcing
3. Caching
4. Deduplication
5. Join with Master Data
6. Enrichment
7. Real-Time Output
8. Batch Reconciliation (Periodic)
9. Consistency Checks
10. Conflict Resolution
11. Historical Data Integrity

