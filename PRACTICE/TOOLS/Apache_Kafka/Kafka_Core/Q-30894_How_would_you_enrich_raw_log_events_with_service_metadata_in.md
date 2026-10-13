---
type: "practice"
difficulty: "Deep"
---

# How would you enrich raw log events with service metadata in a Kafka Streams job before they reach anomaly detection, and how do you co-partition the enrichment KTable?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka and the Lakehouse](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_and_the_Lakehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enrich raw log events with service metadata in a Kafka Streams job, we need to design a process that takes unstructured log data from an input topic, enriches it with relevant metadata (e.g., service name, user ID, timestamp), and then sends the enriched data to another topic for further processing. This enrichment is typically done using KTable operations in Kafka Streams.

#### Triaging the Question

The core of this question lies in understanding how to effectively integrate metadata into log events within a Kafka Streams job. The interviewer will be probing your ability to:

Understand Kafka Streams and its key components (Streams, KStream, KTable).
Implement co-partitioning strategies for efficient data processing.
Design the logic for enriching log events with metadata.

#### Assumptions
We are dealing with a scenario where raw logs are received via an input topic and need to be enriched before being processed by anomaly detection systems.
The service metadata is stored in another Kafka topic or database, which needs to be joined with the log data.
Performance and scalability are critical considerations.
Step-by-Step Solution

Input Stream Setup: We start by reading from an input topic where raw logs are published.

JAVA
KStream<String, String> logStream = builder.stream(INPUT_TOPIC);

Metadata Lookup Table: Create a KTable that stores the service metadata using another Kafka topic or database query.

JAVA
KTable<String, ServiceMetadata> metadataTable = builder.table(METADATA_TOPIC);

Join Operation: Join the log stream with the metadata table to enrich each log event with the relevant metadata.

JAVA
KStream<String, LogEnrichedEvent> enrichedLogStream = logStream.join(
    metadataTable,
    (logKey, logValue, metadata) -> new LogEnrichedEvent(logValue, metadata),
    JoinWindows.of(Duration.ofMinutes(5))
);

Co-Partitioning: Ensure that the logs and metadata are co-partitioned to avoid shuffles during joins.

For this example, let's assume both topics have been produced with the same partition key (e.g., service ID).
If necessary, we can use a custom partitioner or re-partition strategy.

Output: Send the enriched log events to an output topic for further processing.

JAVA
enrichedLogStream.to(ENRICHED_LOG_TOPIC);

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that each log event is enriched with relevant metadata before it reaches anomaly detection. Key steps include setting up the input stream, creating a lookup table for metadata, performing a join operation, ensuring co-partitioning, and directing the output to an appropriate topic.

#### Common Mistakes

Lack of Co-Partitioning: Not aligning the partitions between the log stream and metadata can lead to high shuffle costs.
Ignoring Join Windows: Failing to define proper windows for join operations can result in incorrect data alignment or missed events.
Improper Metadata Lookup: Ensuring that the metadata is up-to-date and correctly structured is crucial, as outdated or incorrectly formatted metadata will negatively impact the enrichment process.
Questions for Further Clarification
How do you ensure the metadata remains up-to-date?
What if there are more than two sources of data (logs and metadata)?
How would you handle large volumes of log data with varying service IDs?

**Comparison:**

| Input Log Stream                | Metadata Table                  | Enriched Log Stream                      |
|---------------------------------|---------------------------------|-----------------------------------------|
| Key: app1, Value: Error log     | Key: app1, Value: {"serviceName": "App1", ...} | Key: app1, Value: Enriched Event with Metadata |
| Key: app2, Value: Info log      | Key: app2, Value: {"serviceName": "App2", ...} | Key: app2, Value: Enriched Event with Metadata |

**Code:**

```java
KStream<String, String> logStream = builder.stream(INPUT_TOPIC);
KTable<String, ServiceMetadata> metadataTable = builder.table(METADATA_TOPIC);

// Joining the stream with the metadata table
KStream<String, LogEnrichedEvent> enrichedLogStream = logStream.join(
    metadataTable,
    (logKey, logValue, metadata) -> new LogEnrichedEvent(logValue, metadata),
    JoinWindows.of(Duration.ofMinutes(5))
).to(ENRICHED_LOG_TOPIC);
```

**Example input / output:**

```text
# Sample input log stream
logStream:
  - key: app1
    value: "2023-10-01T14:00:00Z [ERROR] service:app1 user:admin message:some error occurred"
  - key: app2
    value: "2023-10-01T14:05:00Z [INFO] service:app2 user:guest message:something happened"

# Sample metadata table
metadataTable:
  - key: app1
    value: {"serviceName": "App1", "userId": "admin", "timestamp": 1630790400}
  - key: app2
    value: {"serviceName": "App2", "userId": "guest", "timestamp": 1630876800}

# Enriched log stream after joining
enrichedLogStream:
  - key: app1
    value: LogEnrichedEvent("2023-10-01T14:00:00Z [ERROR] service:app1 user:admin message:some error occurred", {"serviceName": "App1", "userId": "admin", "timestamp": 1630790400})
  - key: app2
    value: LogEnrichedEvent("2023-10-01T14:05:00Z [INFO] service:app2 user:guest message:something happened", {"serviceName": "App2", "userId": "guest", "timestamp": 1630876800})
```

**Data:**

```yaml
# Example service metadata schema
serviceMetadata:
  type: object
  properties:
    serviceName:
      type: string
    userId:
      type: string
    timestamp:
      type: integer

# Sample log messages
logMessages:
  - "2023-10-01T14:00:00Z [ERROR] service:app1 user:admin message:some error occurred"
  - "2023-10-01T14:05:00Z [INFO] service:app2 user:guest message:something happened"
```

**Flow:**

1. Input Log Stream
2. Join with Metadata Table
3. Enrich Log Events
4. Output Enriched Log Stream toTopic

