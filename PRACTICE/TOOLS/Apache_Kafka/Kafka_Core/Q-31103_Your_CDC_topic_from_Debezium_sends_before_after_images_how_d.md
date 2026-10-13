---
type: "practice"
difficulty: "Deep"
---

# Your CDC topic from Debezium sends before/after images; how do you build an upsert into an Iceberg or Delta table that correctly applies inserts, updates, and deletes in order?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Quarantine Topics](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Quarantine_Topics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of building an upsert operation from a Debezium CDC topic into an Iceberg or Delta table using Kafka, we need to ensure that both insertions and updates are handled correctly while preserving transactional integrity. The core idea is to process each event (before/after images) in such a way that it either inserts a new row, updates an existing one, or deletes a row if necessary.

First, let's break down the requirements:

Event Processing: Each message from Kafka contains before and after states of a record.
Inserts: When a row is newly created, we need to insert this into our target table.
Updates: Rows that have changed should be updated in place within the table.
Deletes: For logical deletes (often represented as nullification or marking fields), we must delete the corresponding rows.

We will implement these steps using a combination of Kafka Streams and Iceberg/Delta Lake APIs, ensuring atomicity and consistency across the processing pipeline.

Step-by-Step Solution
1. Setup and Configuration
Kafka Topic: Assume we have a Kafka topic debezium_topic that captures all changes to our database tables.
Iceberg Table Schema: Define an Iceberg table schema that matches your database schema, with appropriate partitioning and sorting if needed.
2. Stream Processing with Kafka Streams
Use Apache Flink or Apache Spark Structured Streaming for stream processing since both frameworks support real-time data processing and integration with Iceberg/Delta Lake.
Define a stream processing job that reads from the Kafka topic, processes each event, and writes to the target Iceberg table.
3. Processing Logic
For each event:
Extract the before and after states.
Determine if it’s an insert, update, or delete operation based on changes in fields like timestamps or primary keys.
Use Iceberg/Delta Lake APIs to perform the corresponding actions:
JAVA
if (event.isInsert()) {
    // Insert new row
    icebergTable.insert(new Row(...));
} else if (event.isUpdate()) {
    // Update existing row
    icebergTable.update(new RowKey(), ...);
} else if (event.isDelete()) {
    // Delete row
    icebergTable.delete(new RowKey());
}
4. Handling Deletes
For logical deletes, set appropriate flags or use tombstones in Iceberg/Delta Lake.
Ensure that the delete operation is idempotent and consistent across multiple processing runs.
5. Error Handling and Retries
Implement retries for failed operations to ensure data integrity.
Use transactional semantics provided by Kafka Streams to handle failures gracefully.

#### Closing Summary

By carefully designing our stream processing logic and leveraging Iceberg/Delta Lake’s capabilities, we can efficiently upsert data from Debezium CDC topics into these modern storage systems. The key is ensuring that each operation (insert, update, delete) is processed correctly while maintaining the integrity of the data lake.

#### Common Mistakes/Follow-Ups
Failure to Handle Deletes: Not properly managing logical deletes can lead to inconsistencies.
Incorrect Event Determination: Misidentifying insertions, updates, and deletions might result in incorrect operations.
Transaction Management: Failing to use transactional semantics can introduce data corruption during failures.

**Comparison:**

```
Before/After States	Action
before: {id: 1, value: 'old_value'}, after: {id: 1, value: 'new_value'}	Update row with id=1 to value='new_value'
```

**Code:**

```java
// Example Kafka Streams Processor for Iceberg Table Updates
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.hadoop.conf.Configuration;
import org.apache.iceberg.Table;

public class DataProcessor {
    private final StreamTableEnvironment tableEnv;
    private final Configuration config;
    private final String topicName;

    public DataProcessor(StreamTableEnvironment tableEnv, Configuration config, String topicName) {
        this.tableEnv = tableEnv;
        this.config = config;
        this.topicName = topicName;
    }

    public void process() {
        // Read from Kafka
        DataStream<String> kafkaStream = ...;

        // Process stream and write to Iceberg Table
        kafkaStream.map(event -> handleEvent(event))
                   .addSink(new IcebergTableWriter(tableEnv, config));
    }

    private Row handleEvent(String event) {
        // Parse event into before/after states
        BeforeAfterStates states = parseEvent(event);

        if (states.isInsert()) {
            return insert(states);
        } else if (states.isUpdate()) {
            return update(states);
        } else if (states.isDelete()) {
            return delete(states);
        }
    }

    private Row insert(BeforeAfterStates states) {
        // Insert logic
        return new Row(...);
    }

    private void update(BeforeAfterStates states) {
        // Update logic
    }

    private void delete(BeforeAfterStates states) {
        // Delete logic using tombstones or flags
    }
}
```

**Example input / output:**

```text
Input Event:
{"before": {"id": 1, "value": "old_value"}, "after": {"id": 1, "value": "new_value"}}

Output Actions:
- Update row with id=1 to value='new_value'
```

**Data:**

```json
{
  "topicName": "debezium_topic",
  "tableSchema": [
    {"name": "id", "type": "int"},
    {"name": "value", "type": "string"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Read Kafka Topic
8. Is Insert?
9. Insert Row
10. Is Update?
11. Update Row
12. Is Delete?
13. Mark for Deletion

