---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd operate a globally-unique monotonic ID service at millions/sec that survives node failure. Compare a single-leader counter batched range allocation and Snowflake-style IDs on availability and ordering.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed ID Generation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_ID_Generation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To operate a globally-unique monotonic ID service capable of handling millions of requests per second (MPS) while ensuring high availability in the face of node failures, we need to compare and contrast two common approaches: single-leader counter batched range allocation and Snowflake-style IDs. Both have their strengths and weaknesses when it comes to availability and ordering guarantees.

#### Triaging the Question

The core of this question is about understanding how to design a distributed system for generating globally-unique monotonic IDs under high load conditions, with a focus on resilience against node failures. The interviewer is probing your knowledge of distributed systems, fault tolerance mechanisms, and performance optimization techniques. We will assume a high-traffic environment where the ID service must handle multiple millions of requests per second.

Single-Leader Counter Batched Range Allocation

#### Overview

Single-leader counter batched range allocation involves one or more centralized servers that generate IDs in batches and distribute them to clients. This approach is simpler but can become a bottleneck under high load, leading to single points of failure.

Key Components
Centralized Leader Node(s): These nodes generate and assign ID ranges.
Client Nodes: Clients request IDs from the leader nodes.
Range Allocation Mechanism: Leaders allocate pre-generated ranges to clients.
Replication Strategy: To ensure high availability, leaders can replicate themselves.
Availability

#### Pros
Simplicity: Easy to implement and understand.
Scalability: Can scale by adding more leader nodes.

#### Cons
Bottleneck: Single-leader architecture can become a bottleneck under high load.
Failure Risk: If the leader node fails, it needs immediate failover or replication.
Ordering

#### Pros
Strict ordering guaranteed if single leader is maintained.

#### Cons
Ordering might be lost during failovers or if multiple leaders exist.
Snowflake-Style IDs

#### Overview

Snowflake-style IDs are designed to provide high scalability and resilience by using a distributed, self-healing approach. These IDs consist of time-based components that ensure uniqueness across nodes without centralized coordination.

Key Components
Time Component: Based on the current timestamp.
Node ID Component: Unique identifier for each node.
Sequence Number Component: Ensures sequential ordering within a short time window.
Availability

#### Pros
Decentralized: No single point of failure as IDs are generated locally by nodes.
High Availability: Nodes can continue to generate IDs even if some fail.

#### Cons
Potential for ID collisions if not managed correctly, although this is extremely rare in practice.
Ordering

#### Pros
Naturally maintains strict ordering based on time and node sequence.

#### Cons
Slight delay due to the need to capture the current timestamp accurately.
Comparative Analysis
Availability
Single-Leader Counter Batched Range Allocation:

#### Pros
Easier to set up, can handle high load with multiple leaders.

#### Cons
Single-leader bottleneck and failure risk.
Snowflake-Style IDs:

#### Pros
High availability and resilience due to decentralized ID generation.

#### Cons
Slightly more complex implementation but inherently avoids single points of failure.
Ordering
Single-Leader Counter Batched Range Allocation:

#### Pros
Strict ordering if the leader remains consistent.

#### Cons
Ordering might be lost during failovers or multiple-leader scenarios.
Snowflake-Style IDs:

#### Pros
Naturally maintains strict ordering with minimal overhead.

#### Cons
No inherent mechanism to handle strict ordering in failure scenarios.

#### Closing Summary

For a globally-unique monotonic ID service that needs to operate at millions of requests per second while maintaining high availability and strict ordering, Snowflake-style IDs are generally the better choice. They leverage distributed, self-healing mechanisms that avoid single points of failure and maintain natural order based on time and sequence.

#### Common Mistakes/Follow-Ups
Mistake: Assuming a simple single-leader architecture will handle millions of requests without considering failover mechanisms.

#### Follow-Up
Discussing specific implementation details such as how to handle edge cases, like ID collisions in Snowflake-style IDs or the impact of network latency on leader selection in batched range allocation.

**Comparison:**

```
Time Component	Node ID Component	Sequence Number Component
Timestamp	Unique Identifier	Sequential Number
```

**Code:**

```python
# Example Python code for generating Snowflake-style IDs
import time

class SnowflakeIDGenerator:
    EPOCH = int(time.mktime((2010, 1, 1, 0, 0, 0, 0, 0, -1)))
    NODE_ID_BITS = 8
    SEQUENCE_BITS = 12
    MAX_NODE_ID = -1 ^ (-1 << NODE_ID_BITS)
    SEQUENCE_MASK = -1 ^ (-1 << SEQUENCE_BITS)

    def __init__(self, node_id):
        self.node_id = node_id
        self.sequence = 0
        self.last_timestamp = -1

    def generate_id(self):
        timestamp = int(time.time() * 1000)
        if timestamp < self.last_timestamp:
            raise Exception("Clock moved backwards. Refusing to generate ID")

        if self.last_timestamp == timestamp:
            self.sequence = (self.sequence + 1) & self.SEQUENCE_MASK
            if self.sequence == 0:
                timestamp = self._wait_for_next_timestamp(self.last_timestamp)
        else:
            self.sequence = 0

        self.last_timestamp = timestamp
        return ((timestamp - self.EPOCH) << (self.NODE_ID_BITS + self.SEQUENCE_BITS)) | \
               (self.node_id << self.SEQUENCE_BITS) | self.sequence
```

**Example input / output:**

```text
generator = SnowflakeIDGenerator(node_id=1)
print(generator.generate_id())  # Output: 1844674407355872001
```

**Data:**

```json
{
  "node_id": 1,
  "sequence": 0,
  "last_timestamp": 1628597320000,
  "timestamp": 1628597321000
}
```

**Flow:**

1. Yes
2. No
3. Generate Timestamp
4. Check Last Timestamp
5. Is Clock Backwards?
6. Throw Exception
7. Update Sequence andGenerate ID
8. Wait for Next Timestamp
9. Set New Timestamp

