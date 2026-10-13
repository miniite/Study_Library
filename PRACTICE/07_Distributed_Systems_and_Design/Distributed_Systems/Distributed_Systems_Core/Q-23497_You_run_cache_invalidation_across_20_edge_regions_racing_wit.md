---
type: "practice"
difficulty: "Deep"
---

# You run cache invalidation across 20 edge regions racing with the DB write, serving stale data for seconds. Walk me through designing invalidation that's correct without killing hit rate.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Across Regions](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a distributed system for cache invalidation that avoids serving stale data while maintaining high hit rates across 20 edge regions, we need to ensure that updates to the database are propagated and acknowledged in such a way that all cache nodes get synchronized without causing unnecessary traffic or latency. This involves understanding how distributed consensus protocols work and leveraging them to achieve consistency.

First, let's define our problem: We have multiple edge regions (e.g., 20 geographically distributed data centers) serving cached reads from a single database write. The goal is to invalidate the cache in such a way that it remains consistent with the database while minimizing the time users experience stale data.

#### Triage

The core of this question revolves around ensuring the correctness and efficiency of cache invalidation across multiple nodes. We need to ensure that all nodes are updated atomically or at least within an acceptable timeframe, without causing undue network load or performance degradation.

#### Assumptions
Database Write: A write operation to the database is performed.
Cache Nodes: Multiple edge regions (20) have cached copies of data.

#### Cons
istency Requirement: We need to ensure that all nodes see consistent updates without serving stale data for too long.
Network Latency: Network latency between any two nodes can be significant due to geographic distribution.

#### Solution

To achieve this, we will use a distributed consensus protocol like Raft or Paxos to ensure that cache invalidation commands are processed correctly across all nodes. The basic steps involve:

Database Write: A write operation is performed on the database.

#### Cons
ensus Protocol: The system uses a consensus protocol to ensure that an update command (invalidate the cache) is executed atomically by all relevant nodes.
Command Propagation: The invalidate command is propagated from the node performing the database write to all other nodes.
Acknowledge: Once a majority of nodes have acknowledged the invalidate command, it is considered complete.
Step-by-Step Solution

Database Write: When a new record is written to the database:

SQL
INSERT INTO customers (id, name) VALUES ('123', 'Alice');

#### Cons
ensus Protocol: The node performing the write uses a consensus protocol like Raft to propose an invalidate command. This command instructs other nodes to clear their cache for the relevant data.

Command Propagation:

A leader is elected in the Raft cluster.
The leader sends the invalidate command to all followers and eventually consistent nodes (a majority of nodes).
Followers and eventually consistent nodes process the command and update their state.

Acknowledge: Once a majority of nodes have acknowledged the invalidate command, it is considered complete:

PYTHON
# Pseudo-code for Raft leader processing an invalidate request
def handle_invalidate_request(command):
    if self.is_leader():
        self.send_to_followers(command)
        await_for_majority_acknowledgements()
        return True
    else:
        return False

Cache Update: Once the invalidate command is acknowledged, all nodes update their cache to reflect the new state.

Monitoring and Retries: Implement a mechanism to handle retries in case of network partitions or other failures. This ensures that even if some nodes do not receive the invalidate command initially, they will eventually catch up when the system converges.

#### Closing Summary

By using a distributed consensus protocol like Raft for cache invalidation, we ensure that all edge regions get consistent updates without serving stale data. This approach minimizes latency and network load while maintaining high hit rates in a distributed environment.

#### Common Mistakes/Follow-Ups
Incorrect Consistency Level: Failing to achieve a majority of acknowledgments can lead to inconsistent states.
Network Partitions: Not handling network partitions properly can result in split-brain scenarios where different nodes have divergent states.
Retry Mechanisms: Overlooking the need for retries and backoff strategies can cause high latency or even data loss.

**Comparison:**

```
Node	State After Write	State After Invalidate
1	Updated	Invalidated
2	Updated	Invalidated
...
20	Updated	Invalidated
```

**Code:**

```python
# Pseudo-code for Raft leader processing an invalidate request
def handle_invalidate_request(command):
    if self.is_leader():
        self.send_to_followers(command)
        await_for_majority_acknowledgements()
        return True
    else:
        return False

# Example of sending a command to followers in Raft protocol
def send_to_followers(command):
    for node in self.followers:
        node.receive_command(command)

# Example of waiting for majority acknowledgments
def await_for_majority_acknowledgements():
    # Wait until more than half of the nodes have acknowledged the command
    pass
```

**Example input / output:**

```text
Input:
- Database write operation: INSERT INTO customers (id, name) VALUES ('123', 'Alice');

Output:
- Consensus protocol command propagation and acknowledgment process.
- Cache invalidation commands sent to all nodes.
- Nodes update their cache state upon receiving valid invalidate command.
```

**Data:**

```yaml
database: customers
fields: id, name
operation: insert
value: {id: '123', name: 'Alice'}
```

**Flow:**

1. Database Write
2. Raft Leader Elected
3. Send Invalidate Command toFollowers
4. Await for MajorityAcknowledgements
5. Cache Update ifAcknowledged
6. Cache Invalidated in All Nodes

