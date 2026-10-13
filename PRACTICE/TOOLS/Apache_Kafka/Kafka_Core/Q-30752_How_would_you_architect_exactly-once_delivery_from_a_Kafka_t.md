---
type: "practice"
difficulty: "Expert"
---

# How would you architect exactly-once delivery from a Kafka topic into Elasticsearch given ES has no transactions? Discuss versioning and optimistic concurrency.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once to External Sinks](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_Beyond_Kafka/Exactly_Once_to_External_Sinks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect exactly-once delivery from a Kafka topic to Elasticsearch, given that Elasticsearch does not support transactions natively, we need to implement custom logic using Apache Kafka's transactional semantics and Elasticsearch’s versioning capabilities. The core idea is to use Kafka transactions for ensuring idempotency and leveraging Elasticsearch's _version field to handle optimistic concurrency control.

This approach involves the following steps:

Producer-Side Transaction Management: Use Kafka producer transactions to ensure that a message is processed exactly once.

#### Cons
umer-Driven Idempotence: Ensure that the consumer logic ensures messages are idempotent, meaning multiple identical requests result in the same outcome as a single request.
Elasticsearch Versioning and Optimistic Concurrency Control (OCC): Use Elasticsearch's _version field to implement OCC to handle conflicts when documents are updated concurrently.

#### Triage

The question is probing your understanding of Kafka transactional semantics, idempotency, and how to integrate these with Elasticsearch. The interviewer might also be interested in the trade-offs involved in implementing this solution, such as performance implications, complexity, and potential failure modes.

#### Assumptions

Kafka version supports transactions.
Elasticsearch cluster can handle the write load.
The application has a mechanism to retry and manage idempotent operations.

#### First-Principles Solution
1. Producer-Side Transaction Management

Kafka’s transactional semantics allow producers to send messages within a transaction, ensuring that all messages in a transaction are processed atomically or not at all. This can be used to ensure exactly-once delivery by grouping related messages together.

Yes
No
Start
Kafka Transaction Begin
Is Message Batch Ready?
Send Messages in Transaction
Kafka Transaction Commit
Retry or Discard
Elasticsearch Index Operation
2. Consumer-Driven Idempotence

The consumer application should be designed to handle duplicate messages gracefully. This can be achieved by maintaining a local state (e.g., in a database) that tracks which documents have been processed and their versions.

Yes
No
Start
Kafka Consumer Fetch
Is Message Duplicate?
Log or Discard
Elasticsearch Index Operation
3. Elasticsearch Versioning and OCC

When performing an index operation, use the document’s _version field to determine if it needs to be updated. This ensures that if two consumers try to update the same document simultaneously, only one will succeed.

No
Yes
Yes
No
Elasticsearch Index Operation
Document Exists?
Create New Document
Check Version
Version Matches?
Update Document
Conflict, Retry or Fail Safely

#### Closing Summary

By combining Kafka's transactional semantics with Elasticsearch’s versioning and optimistic concurrency control (OCC), we can achieve exactly-once delivery. This solution ensures that messages are processed only once and handles conflicts elegantly.

#### Common Mistakes/Follow-Ups
Overlooking Idempotence: Failing to design the consumer logic to handle duplicate messages could lead to data inconsistencies.
Ignoring OCC Failures: Not properly handling version mismatches can result in data loss or corruption.
Scalability Issues: High write load on Elasticsearch may require careful tuning and optimization.

**Code:**

```
Example Kafka producer transaction code

from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['localhost:9092'], transactional_id="my_transactional_id")

with producer.transaction(): future = producer.send('my_topic', b'Hello World')

CODE
Copy

```java
// Example Elasticsearch version check and update
import org.elasticsearch.action.update.UpdateRequest;

UpdateRequest request = new UpdateRequest("index", "type", 1)
                            .doc(jsonBuilder().startObject()
                                    .field("field1", "value1")
                                    .endObject())
                            .upsert(jsonBuilder().startObject()
                                    .field("field2", "default_value")
                                    .endObject());
```

**Data:**

```json
{
  "index": "my_index",
  "type": "_doc",
  "id": "1",
  "version": 3,
  "body": {
    "field1": "value1"
  }
}
```

