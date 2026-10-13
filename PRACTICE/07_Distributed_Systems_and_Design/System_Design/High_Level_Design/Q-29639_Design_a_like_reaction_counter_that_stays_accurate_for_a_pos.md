---
type: "practice"
difficulty: "Deep"
---

# Design a like/reaction counter that stays accurate for a post going viral (10K reactions/sec) without a hot-row problem — how do you aggregate and serve the count?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Counters](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Counters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a like/reaction counter that remains accurate even during high traffic, we need to ensure our system can handle 10K reactions per second (RPS) without encountering a hot-row problem in our database. The key here is to distribute the load and aggregate counts efficiently across multiple servers while maintaining consistency.

The core idea is to use a distributed architecture that leverages both real-time streaming for immediate updates and batch processing for accurate aggregation, all while ensuring fault tolerance and scalability. We'll break down the design into several components: real-time event sourcing, state management in cache, leader election for writes, and periodic background jobs for reconciliation.

**Comparison:**

| Component          | Description                                                                                  |
|--------------------|----------------------------------------------------------------------------------------------|
| Real-Time Stream   | Processes likes in near real-time, updating Redis keys.                                       |
| Cache              | Provides fast read access via Memcached for current like count.                               |
| Database           | Stores historical data and handles reconciliation tasks for accuracy.                        |
| Leader Election    | Ensures only one server writes to the database during peak loads.                             |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Event Source] --> B1[Real-Time Stream Processing]
    A --> C[Database (Redis)]
    B1 --> D[Cache Layer (Memcached/Redis)]
    C --> E[Background Jobs (Elasticsearch/PostgreSQL)]
    F[Leader Election Service] --> G[Write Quorum]
    D --> H[Read Requests (HTTP API)]
```

**Example input / output:**

```text
# Example input-output flow for a like reaction

Input:
- User likes post with ID 1234567890

Output:
- Real-time stream updates Redis key `likes:1234567890` with new count and timestamp.
- Memcached cache is updated with the same data.

Real-time updates are immediately available via HTTP API for read requests, while background jobs periodically reconcile and aggregate counts to ensure accuracy and consistency.
```

**Data:**

```yaml
# Example schema for Redis key-value store
likes:<postId>:
  count: <totalCount>
  lastUpdated: <timestamp>

# Schema for Memcached cache layer
cache:<postId>_like_count: {"count": <totalCount>, "lastUpdated": <timestamp>}
```

**Flow:**

1. User
2. Click Like Button
3. Real-Time Stream Processor
4. Redis Key 'likes:<postId>'updated
5. Memcached Cache Updated
6. HTTP API Read Request
7. Return Current Count fromMemcached
8. Background Job Triggers forPeriodic Reconciliation
9. Database Aggregation andStorage

