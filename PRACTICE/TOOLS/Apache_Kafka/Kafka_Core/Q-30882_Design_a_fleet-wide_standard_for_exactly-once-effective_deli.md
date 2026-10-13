---
type: "practice"
difficulty: "Expert"
---

# Design a fleet-wide standard for exactly-once-effective delivery into heterogeneous sinks (Postgres Elasticsearch Redis S3) each lacking Kafka-native transactions. What common pattern (idempotent upsert, versioning, offset-in-sink) would you mandate, and where must each sink deviate?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Exactly Once to External Sinks](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_Beyond_Kafka/Exactly_Once_to_External_Sinks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a fleet-wide standard for exactly-once-effective delivery into heterogeneous sinks (Postgres, Elasticsearch, Redis, S3), we need to ensure that messages are processed only once and in order, despite the lack of native transactions support in these sinks. The key pattern here is idempotent upsert combined with versioning, along with managing offsets in each sink. This approach ensures data integrity while leveraging the strengths of each storage system.

#### Triaging the Question

The core of this question involves designing a mechanism to achieve exactly-once semantics (EOS) across multiple sinks that do not support native transactions. The interviewer is probing your understanding of distributed systems, data consistency models, and how to adapt standard patterns to fit specific requirements.

#### Assumptions
Exactly-Once Semantics: Ensuring messages are processed only once.
Heterogeneous Sinks: Each sink (Postgres, Elasticsearch, Redis, S3) has its own processing capabilities and limitations.
Idempotent Upsert: Making sure updates can be repeated without changing the state beyond what was intended.
Versioning: Tracking the version of each message to handle conflicts during reprocessing.

#### First-Principles Solution

Message Partitioning:

Messages should be partitioned by a key that uniquely identifies them within the system. This ensures parallel processing without race conditions.

Offset Management:

Maintain a persistent offset store (e.g., Kafka’s own topic or an external database) to track which messages have been processed for each sink.
Each consumer maintains its current offset and updates it atomically after processing a message successfully.

Idempotent Upsert:

Use a combination of upsert operations with versioning in Postgres, Elasticsearch’s _update_by_query, Redis’s SETNX command, or S3’s upload checks.
Versioning ensures that if the same message is received multiple times, only the latest version is applied.

Conflict Resolution:

Implement a retry mechanism to handle retries due to transient failures without duplicating data.
Use logical clocks (e.g., timestamps) or sequence numbers in messages to ensure deterministic processing order.

Error Handling and Retry Logic:

Define a backoff strategy for retries based on the failure type (network, timeout, etc.).
Implement circuit breakers to prevent cascading failures during high traffic.
Common Patterns
Idempotent Upsert: Ensures that operations like inserts or updates are idempotent.
Versioning: Tracks versions of messages to handle retries and ensure consistent state updates.
Deviations by Sink

Postgres:

Use upsert with version numbers: INSERT INTO table (id, value) VALUES (1, 'new_value') ON CONFLICT (id) DO UPDATE SET value = EXCLUDED.value.
Track versions in the message payload to ensure only the latest version is applied.

Elasticsearch:

Use _update_by_query with doc_as_upsert=true: This allows updates based on query conditions, ensuring idempotent operations.
Maintain a version field in documents and use it for conflict resolution.

Redis:

Use SETNX (Set If Not Exists): For simple key-value storage, this ensures that the value is set only once per key.
For more complex scenarios, implement a custom idempotent mechanism using Lua scripts or transactions.

S3:

Use ETags and conditional PUT requests to ensure data integrity during uploads.
Track upload attempts using metadata or separate logs.

#### Closing Summary

The solution involves maintaining consistency across heterogeneous sinks by leveraging the idempotent upsert pattern with versioning, alongside offset management. Each sink requires specific adaptations based on its strengths and limitations. For example, Postgres supports complex upsert logic, while Elasticsearch uses _update_by_query, Redis employs SETNX, and S3 relies on ETags.

#### Common Mistakes/Follow-Ups
Forgetting Offset Management: Ensuring offsets are properly tracked can prevent processing loops or missing messages.
Ignoring Retry Logic: Failing to implement robust retry mechanisms can lead to data inconsistencies or loss.
Not Considering Scalability: Designing the solution without thinking about scaling can result in bottlenecks.

**Comparison:**

```
Topic	Partition	Offset
topic1	partition1	12345
topic1	partition2	67890
```

**Code:**

```python
# Example of a simplified offset store using an external database
import sqlite3

def get_offset(topic, partition):
    conn = sqlite3.connect('offset_store.db')
    cursor = conn.cursor()
    cursor.execute("SELECT offset FROM offsets WHERE topic=? AND partition=?", (topic, partition))
    result = cursor.fetchone()
    return result[0] if result else 0
```

**Example input / output:**

```

Input: process_message("message_id", "data", "topic1-partition1")

Output:

JSON
Copy
{
  "success": true,
  "version": 2,
  "offset": 12346
}
```

**Data:**

```json
{
  "offsets": {
    "topic1-partition1": 12345,
    "topic1-partition2": 67890
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Message
4. Is offset stored?
5. Process message, updateoffset
6. Maintain versioning in storage
7. Store offset and processmessage
8. Update local state and externalsystems

