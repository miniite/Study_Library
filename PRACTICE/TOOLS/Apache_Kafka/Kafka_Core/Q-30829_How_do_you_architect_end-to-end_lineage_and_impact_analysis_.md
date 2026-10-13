---
type: "practice"
difficulty: "Expert"
---

# How do you architect end-to-end lineage and impact analysis so that before anyone changes or deletes a topic, the platform can tell every downstream consumer, stream job, and warehouse table affected? What metadata plane makes this real at fleet scale?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Kafka and the Lakehouse](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_and_the_Lakehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary Overview

When designing an end-to-end lineage and impact analysis system for a Kafka ecosystem, our goal is to track data flows across topics, stream jobs, and downstream warehouses in near real-time. This allows us to identify all dependent components before any changes or deletions are made, ensuring no part of the processing pipeline breaks unexpectedly.

#### Triaging the Question

The question is essentially asking how we can build a robust metadata management system that captures lineage information for Kafka topics. The key aspects include:

Capturing and storing detailed metadata about each topic.
Tracking all consumers and stream jobs downstream from any given topic.
Identifying dependent tables in downstream warehouses.
Ensuring this tracking is scalable, timely, and accurate.

#### Assumptions
and First Principles
Metadata Collection: We need to collect and store metadata for every Kafka topic, including its structure, producers, consumers, and stream jobs that depend on it.
Impact Analysis: The system should be able to quickly determine the impact of changes or deletions by querying this metadata store.
Scalability: With potentially thousands of topics, we need a solution that scales both horizontally and vertically.
Step-by-Step Solution
1. Metadata Plane Design

The metadata plane will act as the central hub for lineage information. We can use a combination of databases and data catalogs to store and manage this metadata:

Topic Metadata: Store topic details, including schema versions, producers, and consumers.
Stream Job Metadata: Capture all stream processing jobs (e.g., Apache Flink, Apache Kafka Streams) that depend on these topics.
Warehouse Table Metadata: Map tables in downstream data warehouses to their source topics.
2. Real-Time Data Ingestion

To ensure real-time lineage and impact analysis:

Kafka Connectors: Use connectors like Confluent Schema Registry or custom connectors to ingest schema changes into the metadata store.

#### Cons
umer Metrics: Monitor Kafka consumer offsets and errors to detect broken pipelines early.
3. Impact Analysis Engine

Develop an engine that can quickly query the metadata store:

Graph Database: Utilize a graph database like Neo4j to model dependencies between topics, stream jobs, and tables. This allows for complex queries about lineage.
Caching Mechanisms: Implement caching to reduce latency in impact analysis.
4. Notification System

Ensure that any changes or deletions trigger notifications to affected downstream systems:

Event Bus: Use a reliable event bus (e.g., Kafka Streams) to broadcast metadata updates and impact alerts.
SLA Monitoring: Ensure the system meets SLAs for timely notifications, even under heavy load.
5. Data Warehouse Integration

Integrate with data warehouses to update dependent tables automatically or via ETL processes:

Incremental Updates: Use incremental updates (delta Lake) to minimize storage costs and latency.
ETL Jobs: Schedule regular ETL jobs that refresh dependent warehouse tables based on topic changes.

#### Implementation
Example
Metadata Schema
Topic

#### Cons
umer
Stream Job
Warehouse Table
Code for Metadata Ingestion (Pseudocode)
PYTHON
# Sample code to ingest metadata into a database
def update_topic_metadata(topic_name, schema_version):
    db = DatabaseConnection()
    query = f"INSERT INTO topics (name, schema_version) VALUES ('{topic_name}', {schema_version})"
    db.execute(query)

def track_consumer(topic_name, consumer_id):
    db = DatabaseConnection()
    query = f"INSERT INTO consumers (topic_name, consumer_id) VALUES ('{topic_name}', {consumer_id})"
    db.execute(query)
Impact Analysis Query
SQL
-- Example SQL to find downstream impact of a topic deletion
WITH dependent_jobs AS (
  SELECT DISTINCT stream_job FROM stream_jobs WHERE topic_name = 'old-topic'
), affected_tables AS (
  SELECT DISTINCT table_name FROM warehouse_tables WHERE source_topic = 'old-topic'
)
SELECT * FROM dependent_jobs UNION ALL SELECT * FROM affected_tables;

#### Closing Summary
+ Common Mistakes

This approach ensures that any change or deletion to a Kafka topic is tracked and its impact on downstream systems is quickly identified. Key challenges include ensuring real-time accuracy, handling large-scale metadata stores efficiently, and maintaining reliability during peak loads.

#### Common mistakes
include:

Overlooking real-time requirements for critical notifications.
Underestimating the complexity of schema changes in dynamic environments.
Neglecting proper caching strategies to handle query performance.

By addressing these points, we can build a resilient and scalable lineage system that supports robust impact analysis.

**Comparison:**

```
Topic Name	Schema Version	Producers	Consumers
user_events	3	producer1, producer2	consumer1, consumer2
Stream Job	Job ID	Name	Topic Name
process_events	job123	process_events	user_events
Warehouse Table	Source Topic	Table Name
user_activity_summary	user_events	user_activity_summary
```

**Example input / output:**

```text
Input: User requests to delete topic 'user_events'
Output:
- Notification sent to consumer1 and consumer2
- Alert generated for stream job process_events
- Metadata updated in database: topic user_events marked as deleted
- Warehouse table user_activity_summary refreshed with latest data
```

**Data:**

```json
{
  "topics": [
    {
      "name": "user_events",
      "schema_version": 3,
      "producers": ["producer1", "producer2"],
      "consumers": ["consumer1", "consumer2"]
    }
  ],
  "stream_jobs": [
    {
      "job_id": "job123",
      "name": "process_events",
      "topic_name": "user_events"
    }
  ],
  "warehouse_tables": [
    {
      "table_name": "user_activity_summary",
      "source_topic": "user_events"
    }
  ]
}
```

**Flow:**

1. Topic Deletion Request
2. Metadata Update
3. Notify Consumers
4. Alert Stream Jobs
5. Refresh Warehouse Tables
6. Consumer Notification Sent
7. Job Alert Generated
8. Warehouse Table Refreshed

