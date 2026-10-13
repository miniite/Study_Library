---
type: "practice"
difficulty: "Expert"
---

# How would you architect a globally distributed write workload — would you reach for Aurora Global Database, Cosmos multi-region writes, or DynamoDB Global Tables, and what conflict-resolution model do you accept in each?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cloud Database Architecture](<../../../../01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Cloud_Database_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a solution for a globally distributed write workload, we need to carefully consider the trade-offs between availability, consistency, partition tolerance (CAP theorem), and cost. In this scenario, we will evaluate Aurora Global Database, Cosmos multi-region writes, and DynamoDB Global Tables based on their features, costs, and suitability.

#### Triaging the Question

The question is essentially asking which cloud database strategy to choose for a globally distributed write workload. The interviewer is probing your understanding of distributed databases, consistency models, and the trade-offs between different architectures in terms of performance, cost, and complexity.

We will make the following assumptions:

The application needs strong consistency or at least sessional consistency.
The system requires low-latency writes to all regions.
Cost is a factor but not the primary concern (though it should be considered).

#### First-Principles Solution
Aurora Global Database

Aurora Global Database is designed for MySQL and PostgreSQL workloads that require global consistency. It provides cross-region replication, allowing for near real-time availability of data across regions.

Key Features:

Cross-region read replicas and a primary region.
Automatic failover to the nearest replica in another region.
Strong consistency guarantees within each region.
Limited support for global transactions (using AWS Global Transaction Manager).

#### Trade-offs

Higher latency compared to local writes due to cross-region communication.
Cost can be higher, especially if multiple regions are used.
Cosmos Multi-Region Writes

Cosmos DB is a globally distributed database service with built-in support for partitioning and multi-master writes. It offers four different consistency models (strong, sessional, bounded staleness, and eventual).

Key Features:

Automatic partitioning and distribution across multiple regions.
Supports global transactions using the Consistency Levels feature.
Multiple consistency levels allow fine-grained control over how globally distributed writes are handled.

#### Trade-offs

Lower latency compared to Aurora due to its distributed nature.
Cost can be higher, depending on the chosen consistency level and data access patterns.
DynamoDB Global Tables

DynamoDB is a fully managed NoSQL database service that supports global tables. It automatically replicates data across multiple regions with low-latency writes.

Key Features:

Automatic cross-region replication for high availability.
Supports strong consistency in read operations.
Can handle large-scale distributed writes efficiently.

#### Trade-offs

Cost is generally lower compared to other managed services like Aurora and Cosmos DB.
Limited transactional support (currently only batch gets and puts).
Conflict Resolution Models
Aurora Global Database

For Aurora, you can choose between strong consistency within a region or use the AWS Global Transaction Manager for cross-region transactions. In cases of conflicts, Aurora will typically resolve them based on the last write wins policy.

Cosmos DB

Cosmos DB supports four consistency models:

Strong Consistency: Ensures that all reads and writes are consistent across regions.
Bounded Staleness: Allows some staleness in data but guarantees it within a user-defined time bound.
Session Consistency: Provides sessional consistency for multi-master write operations, ensuring that subsequent read operations return the latest value written by the same client.
DynamoDB Global Tables

DynamoDB does not natively support complex conflict resolution. However, you can implement custom logic to handle conflicts at the application layer. The default behavior is last write wins.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, Aurora Global Database offers strong consistency but may have higher latency due to cross-region writes. Cosmos DB provides multiple consistency models with varying trade-offs in cost and performance. DynamoDB is generally cheaper and easier to use but lacks native transaction support for complex operations.

For a globally distributed write workload requiring high consistency and low-latency writes, Cosmos DB or Aurora Global Database might be the best choices depending on your specific requirements. If cost is a primary concern, DynamoDB Global Tables could be a viable option with careful handling of conflicts at the application layer.

#### Common Mistakes
Overlooking the importance of consistency models and their implications.
Ignoring the cost implications of different services.
Failing to consider the complexity of implementing custom conflict resolution in DynamoDB.

#### Follow-up
questions might include:

How would you implement conflict resolution if using Cosmos DB with session consistency?
What are the cost differences between these services, and how do they impact your choice?
Can you explain the CAP theorem in relation to these database choices?

**Comparison:**

| Database | Consistency Model         | Replication Latency | Cost |
|----------|---------------------------|---------------------|------|
| Aurora   | Strong                    | Up to 1 second      | High |
| Cosmos DB | Strong / Bounded Staleness| Low                 | Medium - High |
| DynamoDB | Last Write Wins           | Low                 | Low  |

**Code:**

```
MERMAID
Copy
graph TD;
    subgraph "Aurora Global Database"
        A[Aurora Primary]
        B[Beyond Region Replicas]
        C[Global Transaction Manager]
        D[Last Write Wins Conflict Resolution]
        A --> B
        B --> C
        C --> D
    end

    subgraph "Cosmos DB"
        E[Cosmos Primary]
        F[Multiple Regions]
        G[Consistency Levels (Strong, Bounded Staleness)]
        H[Bounded Staleness Conflict Resolution]
        I[Session Consistency Conflict Resolution]
        E --> F
        F --> G
        G --> H
        G --> I
    end

    subgraph "DynamoDB Global Tables"
        J[DynamoDB Primary]
        K[Multiple Regions]
        L[Bounded Staleness Conflict Resolution (Last Write Wins)]
        M[Custom Application Logic for Conflicts]
        J --> K
        K --> L
        K --> M
    end
```

**Example input / output:**

```text
Input:
- Aurora Global Database configuration with strong consistency and cross-region replication.

Output:
- Cross-regional failover within a few seconds.
- Near real-time data availability across regions.

Request:
- Perform write operation in Region A.

Response:
- Operation replicated to other regions in approximately 1 second.

Input:
- Cosmos DB configuration with sessional consistency for multi-master writes.

Output:
- Multi-region replication and automatic partitioning.

Request:
- Client A performs a write operation in Region B.

Response:
- Write acknowledged locally, and the updated data is propagated to other regions according to the chosen consistency model.
```

**Data:**

```json
{
  "AuroraGlobalDatabase": {
    "ConsistencyModel": "Strong",
    "ReplicationLatency": "Up to 1 second"
  },
  "CosmosDB": {
    "ConsistencyModels": ["Strong", "Bounded Staleness"],
    "PartitioningStrategy": "Automatic"
  },
  "DynamoDBGlobalTables": {
    "ConsistencyModel": "Last Write Wins",
    "ReplicationLatency": "Up to 1 second"
  }
}
```

**Flow:**

1. Write Request
2. Primary Region
3. Aurora Replication to OtherRegions
4. Failover Mechanism
5. Aurora Multi-Region Read
6. Write Request
7. Cosmos DB Primary Region
8. Multimaster Write Handling
9. Automatic Partitioning andReplication
10. Bounded Staleness ConflictResolution
11. Write Request
12. DynamoDB Primary Region
13. Replicated to Other Regions
14. Last Write Wins ConflictResolution
15. Custom Application Logic forConflicts

