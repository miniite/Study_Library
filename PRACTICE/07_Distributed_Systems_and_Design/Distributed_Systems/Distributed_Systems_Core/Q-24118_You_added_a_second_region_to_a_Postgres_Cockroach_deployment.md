---
type: "practice"
difficulty: "Deep"
---

# You added a second region to a Postgres/Cockroach deployment for HA but cross-region writes are now slow. Explain the latency tax of cross-region consensus, and how you'd use region-local tables or follower reads to keep the common path fast.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed database system like Postgres or CockroachDB, adding a second region is intended to provide high availability (HA) and redundancy. However, when you enable cross-region writes, there can be significant performance overhead due to the "latency tax" of consensus mechanisms. This answer will explain what the latency tax entails and how we can mitigate it using region-local tables or follower reads.

Firstly, the latency tax refers to the additional time and processing required for distributed systems to ensure data consistency across regions. In a two-region setup, every write operation must be acknowledged by both regions, which introduces network delays and computational overhead. This is because these systems typically use strong consensus algorithms like Raft or Paxos to maintain ACID properties (Atomicity, Consistency, Isolation, Durability) across different nodes.

To mitigate the latency tax while maintaining high availability, we can employ region-local tables or follower reads for the common path of read operations. Region-local tables ensure that data is stored and accessed primarily within a single region, reducing cross-region traffic and improving performance. Follower reads involve setting up "follower" nodes in each region to handle read requests, allowing them to serve local clients without querying the primary node.

Let's break this down step-by-step from first principles:

Understand the Problem:

The current setup has two regions (Region A and Region B).
Cross-region writes are slow due to consensus mechanisms.
We need to balance HA with performance by optimizing read operations.

#### Solution
Approach:

Use region-local tables for common path reads.
Implement follower nodes in each region for handling read requests.

Detailed Explanation:

Region-Local Tables

Region-local tables are designed to store and manage data primarily within a single region. When a client from that region performs a read, the request is handled locally by the table's primary node. This minimizes network latency and avoids the need for cross-region communication.

To implement this:

Create Region-Specific Tables:

For each region, create tables with specific names or prefixes to distinguish them.
Ensure that both regions have access to these tables but only perform writes and reads within their own region.

Database Configuration:

Configure the database to use region-specific sharding or routing rules so that read requests are directed to the appropriate local table.

#### Example
Schema:

SQL
-- Region A Table
CREATE TABLE IF NOT EXISTS sales_region_a (
  id SERIAL PRIMARY KEY,
  product_id INT NOT NULL,
  quantity INT NOT NULL,
  region CHAR(2) DEFAULT 'A' CHECK (region = 'A')
);

-- Region B Table
CREATE TABLE IF NOT EXISTS sales_region_b (
  id SERIAL PRIMARY KEY,
  product_id INT NOT NULL,
  quantity INT NOT NULL,
  region CHAR(2) DEFAULT 'B' CHECK (region = 'B')
);
Follower Reads

Follower reads involve setting up secondary or follower nodes in each region to handle read requests. These followers can be lagging behind the primary node but are capable of serving local client requests without needing to query the primary.

To implement this:

Set Up Follower Nodes:

In each region, set up a follower node that is configured for read-only access.
Configure the database replication settings so that writes still go to the primary nodes, but reads can be served by the followers.

Database Configuration:

Use configuration parameters like read_only or replica_mode in PostgreSQL/CockroachDB to enable follower nodes.
Ensure that client applications are aware of which nodes are followers and direct read requests appropriately.

#### Example
Configuration for CockroachDB:

JSON
{
  "node": {
    "server_options": {
      "replication.auto-join.enabled": true,
      "replication.lease-grant-period": 5s,
      "replication.read-replicas": ["follower_node_1", "follower_node_2"]
    }
  }
}

Client-Side Routing:

Modify client applications to recognize follower nodes and route read requests accordingly.
Use connection pooling or load balancers to efficiently distribute read traffic between primary and follower nodes.

#### Performance Considerations
Read-Write Splitting: Ensure that write operations still go through the primary node in both regions, while reads can be handled by followers.
Synchronization: Regularly sync data from followers back to primaries to maintain consistency, but prioritize performance over perfect synchronization.
Monitoring and Optimization: Continuously monitor read latencies and adjust configurations as needed based on observed behavior.

#### Closing Summary

By implementing region-local tables and follower reads, we can significantly reduce the latency tax associated with cross-region writes while maintaining high availability. This approach ensures that common path reads are handled efficiently within each region, improving overall system performance without compromising data consistency or redundancy.

#### Common Mistakes

Failing to properly configure sharding and routing rules.
Misconfiguring follower nodes, leading to unnecessary network overhead.
Not setting up appropriate monitoring and tuning mechanisms for ongoing optimization.

**Comparison:**

| Region | Table            | Description                                      |
|--------|------------------|-------------------------------------------------|
| A      | sales_region_a   | Stores and manages data for region A             |
| B      | sales_region_b   | Stores and manages data for region B             |

**Code:**

```sql
-- Example SQL for creating region-specific tables
CREATE TABLE IF NOT EXISTS sales_region_a (
  id SERIAL PRIMARY KEY,
  product_id INT NOT NULL,
  quantity INT NOT NULL,
  region CHAR(2) DEFAULT 'A' CHECK (region = 'A')
);

CREATE TABLE IF NOT EXISTS sales_region_b (
  id SERIAL PRIMARY KEY,
  product_id INT NOT NULL,
  quantity INT NOT NULL,
  region CHAR(2) DEFAULT 'B' CHECK (region = 'B')
);
```

**Example input / output:**

```text
-- Example input-output for configuring follower nodes
Input:
{
  "node": {
    "server_options": {
      "replication.auto-join.enabled": true,
      "replication.lease-grant-period": 5s,
      "replication.read-replicas": ["follower_node_1", "follower_node_2"]
    }
  }
}

Output:
{
  "status": "success",
  "message": "Follower nodes configured successfully."
}
```

**Data:**

```json
{
  "node": {
    "server_options": {
      "replication.auto-join.enabled": true,
      "replication.lease-grant-period": 5s,
      "replication.read-replicas": ["follower_node_1", "follower_node_2"]
    }
  }
}
```

**Flow:**

1. Region A Primary
2. Region A Follower
3. Region B Primary
4. Region B Follower
5. Client App
6. Write Operation
7. Primary Node
8. Log Write to Both Regions

