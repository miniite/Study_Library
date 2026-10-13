---
type: "practice"
difficulty: "Deep"
---

# You need change data capture from Postgres to Kafka; compare Debezium reading the logical replication stream against polling an updated_at column, on completeness (deletes) and consistency guarantees.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Replica Lag and Read Your Writes](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of changing Change Data Capture (CDC) from PostgreSQL using Debezium to Kafka, we need to compare two methods: reading logical replication streams with Debezium versus polling an updated_at column. We will focus on completeness and consistency guarantees for both approaches.

#### Triage

The question is essentially asking us to evaluate the pros and cons of these two CDC methods in terms of how they handle deletions (completeness) and ensure data consistency. The core idea here is understanding the underlying mechanisms and their implications.

#### Assumptions
Debezium: Reads logical replication streams, which capture changes made at the database level.
Polling updated_at: Continuously queries rows with an updated_at column to detect changes.
Both methods are being compared in a PostgreSQL environment using Kafka as the downstream system.

#### First-Principles Solution
Debezium Method

Debezium is a popular open-source tool that reads logical replication streams, which provide a more comprehensive view of database changes. These streams include inserts, updates, and deletes, ensuring high completeness and consistency guarantees.

Completeness:

Inserts: Captures all new rows added to the table.
Updates: Tracks only modified columns if specified, otherwise captures entire row.
Deletes: Directly reflects deletions without relying on timestamp comparisons or polling mechanisms.

#### Cons
istency Guarantees:

Debezium ensures that captured events are in the correct order and reflect actual changes as they occur.
Polling updated_at Method

Polling an updated_at column involves regularly querying the table to identify any rows where this timestamp has changed, which can indicate a change. This method relies on periodic polling intervals to detect updates or deletions.

Completeness:

Inserts: Can be missed if no updated_at is updated during insert.
Updates: Only detects changes in the updated_at column; not all columns might have been modified, leading to incomplete data.
Deletes: Not directly captured since deletes do not typically update the updated_at timestamp.

#### Cons
istency Guarantees:

#### Cons
istency is only guaranteed if the polling interval is sufficiently short, but this can lead to increased latency and load on the database.
Potential for lag in detecting changes, as polling happens at fixed intervals rather than real-time.

#### Closing Summary

In summary, Debezium provides a more robust CDC solution by directly consuming logical replication streams, ensuring completeness and consistency. The updated_at method is less reliable due to its reliance on periodic polling, which can lead to incomplete data capture and potential delays in detecting changes.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that the updated_at column will accurately reflect all row-level changes without proper validation.

#### Follow-Up
Discussing alternatives like materialized views or database triggers that could improve consistency when using the updated_at method.
Further Discussion: Exploring how to implement both methods in a production environment and their respective trade-offs.

**Comparison:**

```
Column	Type
id	int
name	string
Event	Operation	Details
123456	INSERT	{"id": 1, "name": "John Doe"}
789012	UPDATE	{"id": 1, "name": "Jane Doe"} (only name changed)
345678	DELETE	{"id": 1}
```

**Code:**

```python
# Example of setting up Debezium with Kafka for CDC

from kafka import KafkaProducer
import json

producer = KafkaProducer(bootstrap_servers=['localhost:9092'],
                         value_serializer=lambda x:
                         json.dumps(x).encode('utf-8'))

def send_debezium_event(event):
    producer.send('my-topic', value=event)
```

**Example input / output:**

```text
Input:
- PostgreSQL table with a logical replication stream enabled

Output:
- Kafka topic containing CDC events for all changes (inserts, updates, deletes)
```

**Data:**

```json
{
  "schema": {
    "type": "struct",
    "fields": [
      {"name": "table_name", "type": "string"},
      {"name": "op", "type": "string"},
      {"name": "ts_ms", "type": "long"},
      {"name": "transaction_id", "type": ["null", "string"]},
      {"name": "source", "type": {"type": "struct", "fields": [
        {"name": "server_id", "type": "int16"},
        {"name": "db", "type": "string"},
        {"name": "table", "type": "string"}
      ]}},
      {"name": "data", "type": ["null", "struct", ...]}
    ]
  },
  "value": {
    "table_name": "my_table",
    "op": "INSERT",
    "ts_ms": 1623098475123,
    "transaction_id": null,
    "source": {"server_id": 1, "db": "my_db", "table": "my_table"},
    "data": {"id": 1, "name": "John Doe"}
  }
}
```

**Flow:**

1. Database Change
2. Debezium
3. Logical Replication Stream
4. Polling Updated At
5. Poll Intervals
6. Updated_at Column Check

