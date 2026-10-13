---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd keep an Elasticsearch/OpenSearch index in sync with a Postgres source of truth using CDC. What at-least-once and ordering guarantees does log-based CDC give, and how do you make the index writes idempotent?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keeping Indexes in Sync via CDC](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Keeping_Indexes_in_Sync_via_CDC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To keep an Elasticsearch/OpenSearch index in sync with a Postgres source of truth using Change Data Capture (CDC), we need to implement a robust system that ensures data consistency and ordering, while also making sure writes are idempotent. The core idea is to capture changes from the Postgres database, transform them into a format suitable for Elasticsearch, and apply these transformations in an ordered manner.

At-a-glance:

Log-based CDC captures changes as they occur in the database.
Idempotency ensures that operations are safe even if they run multiple times.
Ordering guarantees ensure data integrity across distributed systems.

#### Triage
The question is asking about implementing a log-based CDC system to keep Elasticsearch in sync with Postgres, ensuring at-least-once delivery and ordering of writes. We need to understand the mechanisms provided by CDC for consistency and how to make writes idempotent.

#### Assumptions

Both Postgres and Elasticsearch/OpenSearch are up-to-date.
The database changes (INSERT/UPDATE/DELETE) are logged in a changelog table or through a streaming mechanism like Kafka.
The Elasticsearch/OpenSearch index is updated using bulk API for efficiency.

#### First-principles solution

Set Up CDC Mechanism:

In Postgres, use pg_clog (transaction logs), logical decoding, or PgOutput to capture changes in a structured format.
Kafka can be used as the streaming platform to push these changes from Postgres.

Transform and Apply Changes:

Write a consumer that reads from Kafka and transforms the change events into Elasticsearch actions (index, update, delete).
Use Elasticsearch's bulk API for efficient indexing.

Ensure Idempotency:

Design the transformation logic to handle duplicate entries gracefully.
Use transaction IDs or sequence numbers in changes to ensure idempotency.
Implement a deduplication mechanism using Elasticsearch’s _op_type and _version fields.

Ordering Guarantees:

Ensure that the consumer reads from Kafka in the order of change events.
Use ORDER BY on the changelog table if needed to maintain ordering.

Error Handling and Retries:

Implement retry logic for failed writes, with appropriate backoff strategies.
Log errors and track retries to avoid infinite loops.

Monitoring and Validation:

Set up monitoring to ensure data integrity and availability.
Validate the sync process by comparing a sample of records from Postgres and Elasticsearch.

#### Closing Summary

By leveraging CDC mechanisms, we can maintain an Elasticsearch/OpenSearch index in near real-time with Postgres. The key is ensuring idempotency through proper transformation logic and handling ordering guarantees to prevent data corruption. Idempotent writes ensure that operations are safe even if they run multiple times, while ordering ensures data integrity.

#### Common Mistakes/Follow-ups
Not Handling Idempotency Properly: Failing to design idempotent operations can lead to duplicate entries in Elasticsearch.
Ordering Issues: Mismanaging the order of events can result in inconsistencies between Postgres and Elasticsearch.
Error Handling Ignored: Missing robust error handling and retries can cause data loss or corruption.

**Comparison:**

| Change Type | Description                       |
|-------------|-----------------------------------|
| INSERT      | Adds a new document to Elasticsearch.            |
| UPDATE      | Updates an existing document in Elasticsearch.   |
| DELETE      | Marks a document for deletion in Elasticsearch.  |

**Code:**

```python
from kafka import KafkaConsumer, TopicPartition
import json

# Consumer setup
consumer = KafkaConsumer(bootstrap_servers=['localhost:9092'],
                        auto_offset_reset='earliest',
                        enable_auto_commit=False)

for topic in consumer.topics():
    consumer.assign([TopicPartition(topic, 0)])

while True:
    for message in consumer.poll(timeout_ms=1000).values():
        for msg in message:
            change_event = json.loads(msg.value)
            # Transform and apply changes
            apply_change_to_elasticsearch(change_event)
```

**Example input / output:**

```text
Input (Kafka message):
{
  "table_name": "orders",
  "event_type": "INSERT",
  "row_id": 123,
  "data": {"customer_id": 456, "total_amount": 789.0}
}

Output (Elasticsearch bulk API request):
{"index":{"_id":"123"}}
{"customer_id": 456, "total_amount": 789.0}
```

**Data:**

```json
{
  "change_event": {
    "table_name": "orders",
    "event_type": "INSERT",
    "row_id": 123,
    "data": {"customer_id": 456, "total_amount": 789.0}
  }
}
```

**Flow:**

1. Postgres Change
2. Kafka Topic
3. Elasticsearch Bulk API
4. Elasticsearch Index Update
5. Monitoring and Validation
6. Error Handling

