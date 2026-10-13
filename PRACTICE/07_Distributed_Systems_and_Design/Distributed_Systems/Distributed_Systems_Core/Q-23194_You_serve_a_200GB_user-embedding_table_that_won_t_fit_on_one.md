---
type: "practice"
difficulty: "Deep"
---

# You serve a 200GB user-embedding table that won't fit on one node for a recommender. Compare sharding by key, a parameter-server design, and a distributed KV store on latency and hot-key risk, and which you'd pick.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sharding Vector Indexes](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Sharding_Vector_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a 200GB user-embedding table that won't fit on one node in a recommender system, we need to consider how data is managed across multiple nodes to ensure both low latency and mitigate hot-key risk. Let's compare three approaches: sharding by key, parameter-server design, and using a distributed KV store.

#### Triaging the Question

The core of this question lies in understanding the trade-offs between different distributed storage strategies under constraints such as memory limits per node and the need for efficient data retrieval to support high-latency-sensitive operations like real-time recommendations. The interviewer is probing your understanding of distributed systems, latency management, and the handling of hot keys (i.e., keys that are accessed more frequently than others).

#### Assumptions
Data Size: 200GB user-embedding table.
Nodes: Multiple nodes available for storage.
Latency Sensitivity: High sensitivity to latency in serving recommendations.
Hot Keys: Some users or embeddings are accessed more frequently than others.
Sharding by Key

Sharding by key involves distributing the data across multiple nodes based on a hash function applied to keys. For example, if user_id is used as a shard key, you could use a hash function like hash(user_id) % num_nodes.

Latency Analysis
Latency: Low latency for reads and writes since each request is directed directly to the node holding the relevant shard.
Hot Key Risk: High risk if certain keys are accessed disproportionately more frequently. This can lead to hotspots on specific nodes.

#### Implementation
Details
User ID
Shard Key Hash
Cache Hit
Direct Node Access
Parameter-Server Design

In a parameter-server design, the embeddings are stored in shared memory across multiple nodes. Each node has access to all parameters but only processes a subset of them.

Latency Analysis
Latency: Lower latency compared to sharding since nodes can directly access any embedding.
Hot Key Risk: Reduced risk as no single node is overloaded, but still depends on how the system handles data replication and consistency.

#### Implementation
Details
User ID
Parameter Server Access
Cache Hit
Direct Node Access
Distributed KV Store

A distributed key-value (KV) store like Redis Cluster or Cassandra can be used to manage the embeddings. Each embedding is stored as a key, and its value is the user's embedding data.

Latency Analysis
Latency: Depends on network latency between nodes but generally lower for simple key lookups.
Hot Key Risk: Lower risk due to built-in sharding mechanisms that can distribute hot keys across multiple nodes. Replication ensures high availability and fault tolerance.

#### Implementation
Details
User ID
KV Store Lookup
Cache Hit
Direct Node Access
Recommendations

Given the constraints and requirements, sharding by key is generally less suitable due to high risk of hot keys. The parameter-server design offers a good balance but may require complex management for large datasets. A distributed KV store provides a robust solution with built-in mechanisms to handle hot keys and ensure scalability.

#### Closing Summary

For a 200GB user-embedding table, a distributed key-value store is the most suitable choice due to its ability to handle hot keys efficiently and provide low-latency access. While sharding by key offers simple implementation, it poses significant risk in terms of hotspots, and parameter servers require careful management for large datasets.

#### Common Mistakes/Follow-Ups
Ignoring Hot Key Risk: Failing to recognize the importance of handling hot keys effectively.
Overlooking Latency Trade-offs: Not considering the impact on latency when choosing between local node access and networked KV stores.
Inadequate Node Management: Not properly managing node replication and consistency in parameter-server designs.

**Comparison:**

| Strategy | Latency           | Hot Key Risk         |
|----------|------------------|---------------------|
| Sharding by Key | Low per node | High risk of hotspots |
| Parameter-Server | Very low | Replication complexity |
| Distributed KV Store | Medium to low | Built-in sharding and replication |

**Code:**

```python
# Example code for sharding by key (simplified)
def shard_key(user_id, num_nodes):
    return hash(user_id) % num_nodes

# Example for distributed KV store using Redis-like API
import redis

client = redis.Redis(host='localhost', port=6379, db=0)
embedding_key = f"user_{user_id}"
response = client.get(embedding_key)
```

**Example input / output:**

```text
Input:
- User ID: 12345
- Shard Key Hash Function: hash(user_id) % num_nodes

Output for Sharding by Key:
- Node Access: node_3 (assuming 6 nodes and hash(12345) = 3)

Output for Distributed KV Store:
- Network Call: Redis -> Get user_embedding_key (user_{user_id})
```

**Data:**

```json
{
  "user_embedding_table": {
    "total_size": "200GB",
    "data_structure": "key-value pairs of user-embeddings"
  }
}
```

**Flow:**

1. User ID
2. Shard Key Hash
3. Cache Hit
4. Direct Node Access
5. Parameter Server Access
6. Cache Hit
7. Direct Node Access
8. KV Store Lookup
9. Cache Hit
10. Direct Node Access

