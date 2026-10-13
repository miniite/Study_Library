---
type: "practice"
difficulty: "Deep"
---

# Design a global user-identity/auth store on Spanner that must be strongly consistent worldwide with sub-100ms local reads; how do you reconcile that with the write-consensus latency floor?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Query Execution](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Distributed_Query_Execution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a global user-identity/auth store using Google Cloud Spanner that achieves strong consistency globally while ensuring sub-100ms local reads, we need to carefully balance the trade-offs between strong consistency and write-consistency latency. This problem is akin to choosing between a perfect world where everyone always has the latest information immediately, or a slightly less perfect but more efficient one. Let’s break down how we can achieve this using Spanner's features.

Firstly, let’s clarify what the interviewer is probing:

Understanding of Distributed Systems and Consistency Models: Spanner uses a distributed database model that supports both strong consistency (read-your-writes) and eventually consistent models.
Latency Considerations: We need to ensure sub-100ms local reads while dealing with the inherent latency challenges in maintaining global consistency.

#### Assumptions
The system will have multiple regions, as Spanner is designed for multi-regional deployments.
Reads are more frequent than writes.
There are no hotspots or high write contention areas that could cause significant delays.
The application can tolerate eventual consistency for some operations but not for user authentication.
Step-by-Step Solution

Global Schema Design:

Use Spanner’s multi-region capabilities to ensure data is replicated across regions, providing global availability and strong read-your-writes consistency.
Design the schema with partition keys that distribute writes evenly across regions to avoid hotspots.

Write Consistency Mechanism:

Implement a distributed commit mechanism to handle write operations globally. This involves using Spanner’s transactional capabilities to ensure data integrity during writes.
For example, use a global transaction for creating or updating user identities, ensuring all changes are applied in a coordinated manner across regions.

Read Mechanism:

Implement local read optimizations by leveraging Spanner's locality feature. When a user request comes from within the same region as their data, prefer to serve it locally.
For cross-region reads, use global reads, which will fetch the latest committed data but may have slightly higher latency.

Local Read Optimization:

Use Spanner’s local read consistency mode for operations that can tolerate some staleness. This allows reads from within a region without incurring the cost of a global read.
For example, if a user logs in locally, fetch their data using the local read path to ensure sub-100ms response times.

Write Latency Management:

Implement retries and backoff strategies for write operations to handle network latency and ensure eventual success.
Use Spanner's multi-version concurrency control (MVCC) to manage concurrent writes without conflicts, ensuring strong consistency in subsequent reads.

#### Closing Summary

By leveraging Spanner’s capabilities for multi-region replication and transactional integrity, we can design a globally consistent user-identity/auth store that provides sub-100ms local read performance. The key is balancing the need for global consistency with efficient local read operations through careful schema design, distributed transactions, and localized data access strategies.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring write latency in favor of strong consistency, leading to poor user experience.

#### Follow-Up
Discuss how to handle scenarios where writes are delayed due to network issues or high contention. Consider implementing caching layers or distributed queues to buffer and prioritize critical writes.

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant User as U
    participant RegionA as R1
    participant RegionB as R2
    participant Spanner
    U->>R1: Request Local Read
    R1->>Spanner: Fetch Data Locally
    Spanner-->>R1: Data (Sub-100ms)
```

**Example input / output:**

```text
Input:
- User logs in from RegionA

Output:
- Local read request sent to DC1 of RegionA
- Data fetched and authenticated within sub-100ms response time
```

**Data:**

```json
{
  "regions": [
    {
      "name": "RegionA",
      "datacenters": ["DC1", "DC2"],
      "replicas": [
        {"id": 1, "dc": "DC1"},
        {"id": 2, "dc": "DC2"}
      ]
    },
    {
      "name": "RegionB",
      "datacenters": ["DC3", "DC4"],
      "replicas": [
        {"id": 3, "dc": "DC3"},
        {"id": 4, "dc": "DC4"}
      ]
    }
  ],
  "schema": {
    "users": {
      "columns": [
        { "name": "user_id", "type": "INT64" },
        { "name": "email", "type": "STRING(255)" },
        { "name": "password_hash", "type": "BYTES" }
      ],
      "primary_key": ["user_id"]
    }
  }
}
```

