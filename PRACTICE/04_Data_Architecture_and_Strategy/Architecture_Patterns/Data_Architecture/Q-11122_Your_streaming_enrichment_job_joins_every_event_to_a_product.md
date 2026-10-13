---
type: "practice"
difficulty: "Deep"
---

# Your streaming enrichment job joins every event to a product-catalog reference table. Where do you cache the reference and what happens to in-flight events when the catalog updates mid-stream?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ End to End No Data Loss](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to consider where to cache the product-catalog reference data for efficient lookups during streaming enrichment jobs. Additionally, we must think about how to handle real-time catalog updates without disrupting ongoing processing of in-flight events.

#### Triaging the Question

The core of the question is twofold:

Caching Strategy: Determining where and how to cache the product-catalog reference data for optimal performance.
Real-Time Updates Handling: Managing the synchronization between the cached catalog and live updates to ensure that in-flight events are not affected.

#### Assumptions
The streaming job processes a high volume of events.
The product-catalog is relatively small compared to the event stream volume.
There might be periodic, infrequent, or frequent updates to the catalog.
The streaming data source and reference data storage support real-time querying.

#### First Principles Solution
Caching Strategy

Caching the product-catalog in a distributed cache like Redis or an in-memory data grid (IMDG) can provide fast lookups. This approach ensures that each event can be enriched quickly without making expensive database queries for every lookup.

Distributed Cache: Store the catalog in a distributed cache like Redis, which supports very low latency access and can handle high concurrency.
In-Memory Data Grid (IMDG): Use an IMDG like Hazelcast or Coherence, which not only caches data but also provides advanced features such as data replication and partitioning to ensure high availability.
Real-Time Updates Handling

To manage real-time updates without disrupting in-flight events:

Delta Updates: Apply incremental changes (deltas) from the catalog update to a delta store.
Synchronization Mechanism: Use a synchronization mechanism like Apache Kafka or AWS Kinesis Data Streams to broadcast catalog updates to all relevant cache nodes.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, caching the product-catalog in a distributed cache is crucial for performance. Simultaneously, handling real-time updates requires a delta update strategy and proper synchronization mechanisms to ensure that in-flight events are not affected.

#### Common Mistakes

Failing to consider real-time updates and their impact on processing.
Not using a distributed cache, leading to slower lookups.
Ignoring the need for delta updates and full re-caching, which can introduce unnecessary downtime or data inconsistencies.

#### Follow-Ups

Discuss how to handle large catalog sizes if the reference data becomes significantly larger than initially expected.
Explore alternative strategies like periodic batch processing of events to update cache entries.

#### Cons
ider edge cases where the catalog might become too large for in-memory storage, requiring a hybrid approach with on-disk storage as well.

**Comparison:**

```
Cache Strategy	Redis	In-Memory Data Grid (IMDG)
Latency	Low	Very Low
Concurrency	High	High
Replication	Limited	High
Partitioning	Basic	Advanced
```

**Code:**

```python
# Example Python code snippet using Redis for caching
import redis

cache = redis.Redis(host='localhost', port=6379, db=0)

def get_product_info(event_id):
    catalog_key = f"catalog:{event_id}"
    if not cache.exists(catalog_key):
        # Fetch from the database or other persistent storage
        result = fetch_from_db(event_id)
        cache.set(catalog_key, result, ex=3600)  # Cache for an hour
    return cache.get(catalog_key)
```

**Example input / output:**

| Event (Input) | Cached Catalog Entry (Output) |
|---------------|------------------------------|
| {<br>id: "12345",<br>timestamp: "2023-10-01T10:00:00Z"<br>} | {"product_id": "12345", "name": "Product A", "price": 9.99} |

**Data:**

```json
{
  "event": {
    "id": "12345",
    "timestamp": "2023-10-01T10:00:00Z"
  },
  "catalog_entry": {
    "product_id": "12345",
    "name": "Product A",
    "price": 9.99
  }
}
```

**Flow:**

1. Yes
2. No
3. Event Stream
4. Cache Lookup
5. Cache Hit?
6. Retrieve Catalog Data fromCache
7. Fetch from Persistent Storage
8. Cached in Memory
9. Catalog Update
10. Delta Store
11. Cache SynchronizationMechanism
12. Cached Entries Updated

