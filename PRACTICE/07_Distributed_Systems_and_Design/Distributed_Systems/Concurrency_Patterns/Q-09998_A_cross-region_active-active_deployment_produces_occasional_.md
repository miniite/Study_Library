---
type: "practice"
difficulty: "Expert"
---

# A cross-region active-active deployment produces occasional lost updates on the same logical record edited in two regions. Walk through whether you fix this with last-writer-wins, CRDTs, or region-affinity routing, and how you'd explain the "some concurrent edits will conflict" reality to product and support.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-service Invalidation Races](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When dealing with a cross-region active-active deployment that experiences lost updates due to concurrent edits, several concurrency patterns can be considered: last-writer-wins (LWW), conflict-free replicated data types (CRDTs), and region-affinity routing. Each approach has its pros and cons, and the choice depends on the specific requirements of the system. The reality is that some concurrent edits will inevitably conflict, which must be communicated clearly to both product and support teams.

#### Triaging the Question

The question revolves around ensuring data consistency in a cross-region active-active deployment where multiple regions can independently edit the same record. The primary objective is to minimize or eliminate lost updates due to race conditions. We need to evaluate three main strategies: last-writer-wins (LWW), CRDTs, and region-affinity routing.

#### Assumptions
Latency: Network latency between regions.
Concurrency Level: Frequency of concurrent edits in each region.
Data Sensitivity: How critical it is for updates to be processed exactly as intended.

#### First Principles Solution
Last-Writer-Wins (LWW)

Concept: In LWW, the last entity that writes to a record wins. This approach is straightforward and minimizes complexity but can lead to data loss if multiple writers update the same record simultaneously.

#### Implementation

Implement a timestamp-based mechanism where each write includes a unique identifier or timestamp.
On read, return the value with the latest timestamp.

#### Pros

Simple to implement.
Low overhead since it relies on timestamps rather than complex logic.

#### Cons

May result in data loss if two concurrent writes happen at nearly the same time.
Can lead to unpredictable outcomes and stale data due to network latency.
Conflict-Free Replicated Data Types (CRDTs)

Concept: CRDTs are designed to handle concurrent updates without coordination. They ensure that any sequence of valid operations applied in any order results in a consistent state.

#### Implementation

Use CRDTs like Set, Counter, Map, or Vector Clock for different data structures.
Implement merge functions that resolve conflicts automatically during replication.

#### Pros

High fault tolerance and resilience to network partitions.

#### Cons
istent behavior across all nodes.

#### Cons

Complex implementation and understanding required.
May introduce overhead due to the need for custom conflict resolution logic.
Region-Affinity Routing

Concept: In region-affinity routing, write operations are routed to a specific region where the data is currently stored. Reads can be handled from any region but writes always go to the primary region.

#### Implementation

Use a distributed hash table or similar mechanism for determining the primary and secondary regions.
Ensure that read/write consistency is maintained by coordinating between regions.

#### Pros

Ensures strong consistency since write operations are directed to one region only.
Simpler to implement compared to CRDTs.

#### Cons

Introduces complexity in managing region affinity and ensuring availability.
Requires careful coordination between regions to maintain consistency.

#### Closing Summary

Each approach has trade-offs. LWW is simple but risks data loss, CRDTs are complex but ensure consistent behavior, and region-affinity routing maintains strong consistency at the cost of additional complexity. The choice depends on the specific use case and requirements.

#### Common Mistakes/Follow-Ups
Forgetting to consider network latency: Network delays can significantly impact LWW's performance.
Overlooking edge cases in CRDTs: CRDTs must handle various concurrency scenarios, such as merge conflicts during replication.
Neglecting region affinity routing coordination: Ensuring consistent writes across regions is crucial.
Explanation for Product and Support

To explain the reality of concurrent edits to product and support teams:

"Concurrency issues are a common challenge in distributed systems. Even with best practices, some data updates will conflict due to network latency or race conditions. To address this, we can use several strategies:

Last-Writer-Wins: Simple but risks losing updates.
CRDTs: More complex but ensures consistent behavior and high fault tolerance.
Region-Affinity Routing: Ensures strong consistency but introduces complexity.

We need to carefully evaluate these trade-offs based on the criticality of data and performance requirements. In all cases, some conflicts will occur, and we must be prepared for them."

**Comparison:**

```
Strategy	Simplicity	Consistency	Complexity
LWW	High	Low	Low
CRDTs	Low	High	Medium
Region-Affinity	Medium	High	Medium to High
```

**Code:**

```python
# Example LWW implementation in Python
class LastWriterWins:
    def __init__(self):
        self.data = {}

    def write(self, key, value, timestamp):
        if key not in self.data or self.data[key]['timestamp'] < timestamp:
            self.data[key] = {'value': value, 'timestamp': timestamp}

    def read(self, key):
        return self.data.get(key, {}).get('value', None)
```

**Example input / output:**

```text
Input (us-east-1):
Write key: userProfile, value: Jane Doe, timestamp: 1630452020

Output (us-east-1):
Last Writer Wins: Jane Doe

Input (eu-west-1):
Read key: userProfile

Output (eu-west-1):
Last Writer Wins: Jane Doe
```

**Data:**

```json
{
  "key": "userProfile",
  "writes": [
    {"region": "us-east-1", "value": "John Doe", "timestamp": 1630452000},
    {"region": "eu-west-1", "value": "Jane Smith", "timestamp": 1630452010}
  ]
}
```

**Flow:**

1. User writes
2. LWW Writes
3. Reads from any region
4. Timestamp check on write
5. User writes
6. CRDT Writes
7. Merge during read
8. Auto-conflict resolution
9. User writes
10. Region affinity routing
11. Send to primary region
12. Distribute reads across regions

