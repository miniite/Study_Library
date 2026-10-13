---
type: "practice"
difficulty: "Deep"
---

# Design near-real-time index updates so a price or stock change is searchable within seconds across a 100M-doc index — how do you propagate updates without full reindex and handle the read-during-refresh window?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Price Feeds](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Price_Feeds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design near-real-time index updates in a system with a massive document index, our goal is to ensure that changes to documents (such as price or stock updates) are searchable within seconds without triggering a full reindex of all 100 million documents. This involves designing a robust and scalable architecture that can handle the propagation of these updates efficiently while ensuring minimal disruption during the refresh window.

The key idea here is to decouple the update process from the search index, allowing for incremental updates rather than complete rebuilds. We need to focus on minimizing downtime and ensuring high availability during the index refresh process. Let's break down the solution step-by-step:

Understand the Requirements:

We need real-time or near-real-time propagation of document changes.
The system should allow searches to be performed without a full reindex.
Downtime must be minimal and predictable.

#### Assumptions

Documents are added, updated, or deleted frequently but not excessively.
We can tolerate some delay in indexing for critical updates (e.g., 5-10 seconds).
The system should handle the read-during-refresh window seamlessly.

#### Solution
Outline:

Use a message queue to decouple the update process from the index refresh.
Implement an incremental reindexing strategy.
Design a mechanism for handling reads during the refresh period.

Step-by-Step Solution:

1. Decoupling with Message Queue

We will use a distributed messaging system like Kafka or RabbitMQ to decouple the document updates from the indexing process. This ensures that changes are first sent to the queue and then processed by the indexing service at its own pace.

Document Update
Queue Producer
Kafka/RabbitMQ Queue
Queue Consumer
Indexing Service
2. Incremental Reindexing Strategy

Instead of reindexing the entire 100 million documents, we will only index new or changed documents. This can be achieved using a combination of document versioning and change detection.

Document Versioning: Each document maintains a version number.
Change Detection: A background process checks for changes in the document store and queues these changes for indexing.
Index Versioning: Maintain an index version that tracks which documents have been indexed and their versions. This allows us to selectively reindex only the changed or new documents.
3. Handling Reads During Refresh

To handle reads during the refresh period, we can implement a hybrid search mechanism:

Hybrid Search Engine: Use a combination of a primary index (e.g., Lucene) for quick searches and a secondary index (e.g., a simple hash table or in-memory store) for faster lookups.
Read Locks: Temporarily lock reads during the refresh period to ensure consistency. This can be done by maintaining a read-write lock on the secondary index.
4. Failure Modes and Considerations
Message Queue Reliability: Ensure the message queue is highly available and fault-tolerant.
Backpressure Handling: Implement backpressure mechanisms in the indexing service to handle spikes in updates.
Error Handling: Handle errors gracefully, ensuring that failed updates do not affect the overall system.

#### Closing Summary

By using a distributed messaging system for decoupling, implementing an incremental reindexing strategy, and handling reads during refresh with a hybrid search engine, we can ensure near-real-time index updates without full reindex. This design minimizes downtime and ensures high availability, making it suitable for a 100 million document index.

#### Common Mistakes/Follow-Ups
Failing to Use Message Queues: This can lead to direct coupling of the update process with indexing, causing performance issues.
Ignoring Read Locks: Failing to handle reads during refresh periods can result in inconsistent search results.
Inadequate Backpressure Handling: Without proper backpressure mechanisms, spikes in updates can overwhelm the system.

**Comparison:**

| Document ID | Version |
|-------------|---------|
| 1           | 5       |
| 2           | 3       |

**Code:**

```python
# Example of a simple message producer function using Kafka
from kafka import KafkaProducer

def produce_to_kafka(topic, document_change):
    producer = KafkaProducer(bootstrap_servers=['localhost:9092'])
    future = producer.send(topic, value=document_change)
    result = future.get(timeout=60)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Document Change Message (`{"document_id": 2, "new_version": 4}`) sent to Kafka Queue | Message Queued and Consumed by Indexing Service |

**Data:**

```json
{
  "documents": [
    {"id": 1, "title": "Doc A", "version": 5},
    {"id": 2, "title": "Doc B", "version": 3}
  ],
  "changes": [{"document_id": 1, "new_version": 6}]
}
```

**Flow:**

1. Document Update
2. Queue Producer
3. Kafka/RabbitMQ Queue
4. Queue Consumer
5. Indexing Service
6. Indexing Service
7. Update Primary Index
8. Update Secondary Index(Optional)
9. Read Request
10. Hybrid Search Engine
11. Check Primary Index forDocument Existence
12. If Not Found, Check SecondaryIndex

