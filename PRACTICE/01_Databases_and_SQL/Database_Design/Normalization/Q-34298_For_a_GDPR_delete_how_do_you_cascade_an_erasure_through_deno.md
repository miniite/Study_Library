---
type: "practice"
difficulty: "Expert"
---

# For a GDPR delete, how do you cascade an erasure through denormalized copies that have spread into search indexes caches and analytics extracts?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Denormalization in Lakes](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with cascading a data deletion in a distributed system involving normalized and denormalized data across search indexes, caches, and analytical extracts, it's crucial to ensure both consistency and efficiency. The core challenge is to propagate the deletion signal through various layers of storage without introducing race conditions or performance bottlenecks.

#### Triage
The question asks how to cascade an erasure (data deletion) in a GDPR context through denormalized copies that have spread across search indexes, caches, and analytics extracts. This involves understanding the current data architecture and devising a strategy that minimizes the risk of inconsistencies while ensuring compliance with regulatory requirements like GDPR.

Key Assumptions:

The system uses both normalized and denormalized storage.
Search indexes, caches, and analytics are used for performance optimization.
There is a need to ensure data integrity and consistency across all layers.

Step-by-Step Solution:

Identify the Target Record: Begin by identifying which record needs to be deleted due to GDPR compliance. This could involve querying the normalized database or using unique identifiers.

Signal Generation: Generate a delete signal for this record in the normalized storage layer. Use appropriate logging mechanisms like Apache Kafka or AWS SNS/SQS to ensure that this signal is propagated reliably and asynchronously.

Cache Invalidate: Invalidate the cache entries associated with the target record. This might involve sending messages to distributed caches (like Redis) using a pub/sub pattern, ensuring they expire or are invalidated based on the delete event.

Search Index Update: Update the search indexes by removing the target document. Elasticsearch and other full-text search engines provide APIs for deleting documents. Ensure that these deletions are handled asynchronously to avoid blocking other operations.

Analytics Data Deletion: Identify any analytics extracts or aggregated data that might contain references to the deleted record. This could involve querying data lakes, Hadoop clusters, or cloud storage services like Amazon S3 and performing batch updates to remove relevant records.

Metadata Updates: Update metadata associated with the deleted record in both the normalized database and other layers. This helps in tracking deletions and ensures that future queries are aware of the changes.

Audit Logging: Maintain a detailed audit log of all actions taken during the cascading deletion process. This is crucial for compliance, as it allows you to prove what was done, when, and by whom.

#### Closing Summary
To cascade an erasure through denormalized copies in a GDPR context, it’s essential to have a well-defined strategy that covers multiple layers: normalized storage, caches, search indexes, and analytics. This involves generating signals, updating various systems asynchronously, and maintaining audit logs for compliance.

#### Common Mistakes/Follow-Ups

Race Conditions: Ensure that all layers of the system are updated atomically or in coordination to avoid race conditions.
Performance Impact: Monitor the performance impact on caches and search indexes during cascading deletions.
Data Integrity: Verify that data integrity is maintained across different systems, especially when using distributed architectures.

**Comparison:**

```
Layer	Action
Normalized	Generate delete signal
Cache	Invalidate cache entries
Search	Delete document from index
Analytics	Update aggregations and metadata
```

**Code:**

```
Example of generating a delete signal for a normalized database

import kafka

def generate_delete_signal(record_id): producer = kafka.KafkaProducer(bootstrap_servers='localhost:9092') message = f"DELETE {record_id}" producer.send('deletion-topic', value=message.encode())

CODE
Copy

```python
# Example of cache invalidation using Redis pub/sub pattern
import redis

def invalidate_cache(record_id):
    r = redis.Redis(host='localhost', port=6379, db=0)
    channel = f"cache:{record_id}"
    r.publish(channel, 'INvalidate')
```

**Example input / output:**

```text
Input:
- Record ID: 12345

Output:
- Cache invalidated for record 12345
- Search index document removed for record 12345
- Analytics data updated to remove references to record 12345
```

**Data:**

```json
{
  "record": {
    "id": "12345",
    "name": "John Doe",
    "email": "john.doe@example.com"
  }
}
```

**Flow:**

1. Identify Target Record
2. Generate Delete Signal
3. Cache Invalidate
4. Search Index Update
5. Analytics Data Deletion
6. Metadata Updates
7. Audit Logging

