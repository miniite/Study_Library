---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd store deduplicated 'latest state per entity' derived from an append-only event log so a service can look it up quickly. What are your serving-layer options?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store and efficiently query the "latest state per entity" from an append-only event log, we need to design a robust data architecture that balances storage efficiency with fast lookup capabilities. This scenario involves two primary components: the storage layer (event log) and the serving layer for quick lookups.

#### Triage

The question is essentially about how to efficiently store derived state information in such a way that it can be quickly queried, while ensuring that the data remains consistent and up-to-date with the latest state of each entity. The key aspect here is balancing storage efficiency (to avoid unnecessary duplication) and query performance (for quick lookups).

#### First-Principles Solution
Storage Layer: Event Log

The event log itself will store append-only events, which means new data is added but old data cannot be modified or deleted. Each event contains the entity ID and its state at that moment.

Serving Layer Options
Deduplication and State Aggregation: Store only the latest state per entity within the event log by maintaining a pointer (e.g., timestamp, sequence number) to the most recent event for each entity.
Materialized Views or Summary Tables: Create a separate table that contains the latest state of each entity based on the events in the log. This table can be derived from the event log and kept up-to-date through a scheduled process (e.g., incremental updates).
Detailed Solution
Step-by-Step Reasoning

Event Log Schema

entity_id (string): Identifier for the entity.
state (JSON): The current state of the entity.
timestamp (datetime): Timestamp when the event was recorded.

Materialized View Approach

Create a table named latest_state_view that contains the latest state per entity.
Use SQL to periodically populate this view by selecting the most recent record for each entity from the event log.

Serving Layer Architectures

Database-Based Serving Layer (Relational or NoSQL)

#### Pros
Direct query capabilities leveraging database indexes and caching mechanisms.
Easy integration with existing BI tools.

#### Cons
Potential write amplification due to periodic updates.
Complexity in handling large volumes of writes.

Cache-Based Serving Layer (e.g., Redis, Memcached)

#### Pros
Extremely fast read performance due to cache hits.
Can be updated asynchronously with a background job.

#### Cons
Requires additional infrastructure and maintenance for caching.
Potential stale data if the cache misses.

Hybrid Approach (Database + Cache)

Use a database as the primary store and a cache to serve read requests. Update the cache periodically or asynchronously with new events.

#### Closing Summary

By combining an append-only event log with a materialized view or summary table, we can efficiently store and query the latest state per entity. The choice of serving layer depends on the specific requirements for performance, scalability, and maintainability. A database-based approach is suitable for consistent data access and direct querying, while a cache-based approach excels in read-heavy scenarios with less frequent writes.

#### Common Mistakes/Follow-Ups
Ignoring Event Log Structure: Failing to consider how events are structured can lead to inefficient queries or incorrect deduplication.
Overlooking Cache Warm-Up: Not properly warming up the cache can result in initial high latency for read requests.
Focusing Only on Performance: Ignoring data consistency and correctness might lead to stale or inaccurate state information.

**Comparison:**

```
Entity ID	Name	Email
user123	John Doe	john.doe@example.com
```

**Code:**

```sql
-- Event Log Schema Example
CREATE TABLE event_log (
    id SERIAL PRIMARY KEY,
    entity_id VARCHAR(255),
    state JSONB,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Materialized View Creation
CREATE MATERIALIZED VIEW latest_state_view AS
SELECT
    entity_id,
    MAX(timestamp) AS last_updated,
    state
FROM event_log
GROUP BY entity_id;
```

**Example input / output:**

```text
-- Example SQL Query to Get Latest State
SELECT
    entity_id,
    state->>'name' AS name,
    state->>'email' AS email
FROM latest_state_view
WHERE entity_id = 'user123';
```

**Data:**

```json
{
  "entity_id": "user123",
  "state": {
    "name": "John Doe",
    "email": "john.doe@example.com"
  },
  "timestamp": "2023-10-01T12:00:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Event Log
4. Is latest state needed?
5. Maintain Materialized View
6. Direct Query from Event Log
7. Update Cache Asynchronously

