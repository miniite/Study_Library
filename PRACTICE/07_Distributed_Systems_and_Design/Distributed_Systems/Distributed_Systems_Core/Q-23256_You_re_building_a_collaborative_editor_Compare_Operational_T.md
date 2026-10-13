---
type: "practice"
difficulty: "Expert"
---

# You're building a collaborative editor. Compare Operational Transform and CRDTs for this. What are the correctness and engineering trade-offs, and what happens to tombstone growth over time?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event Schema Evolution](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compare Operational Transformation (OT) and Conflict-free Replicated Data Types (CRDTs) in a collaborative editor context, we need to understand their core principles, correctness guarantees, engineering trade-offs, and long-term implications on tombstone growth.

Operational Transformation is based on transforming local edits into operations that can be applied to other replicas of the document. CRDTs, on the other hand, ensure eventual consistency without requiring explicit conflict resolution by allowing multiple updates to coexist and merge automatically.

Let's break down these concepts from first principles:

#### Triage
Question Breakdown
OT vs. CRDTs: Compare two approaches for achieving eventual consistency in a collaborative editor.
Correctness: Evaluate the correctness guarantees provided by each method.
Engineering Trade-offs: Discuss the trade-offs between OT and CRDTs, particularly focusing on complexity, implementation effort, and performance.
Tombstone Growth: Analyze how tombstones (undesired state) grow over time in both methods.

#### Assumptions
Both systems operate within a distributed network with variable latency.
Replicas may become temporarily out of sync due to network partitions or server failures.
The goal is to ensure that all replicas converge to the same state eventually, without losing data or generating conflicting states.
Step-by-Step Solution
Operational Transformation (OT)

Key Idea: OT transforms local edits into a sequence of operations that can be applied to other replicas. This ensures that every replica can apply these operations in any order and still end up with the correct state.

Correctness Guarantees: OT provides strong consistency guarantees, ensuring that all replicas converge to the same state as if they were operating on a single machine. However, this comes at the cost of complexity and potentially high computational overhead for transformation.

Engineering Trade-offs

Complexity: Implementing OT requires handling complex transformations of edits.
Performance: Transformations can be computationally expensive, especially for large documents or frequent operations.
Scalability: Scales well but may introduce latency due to the need to apply transformations.
Conflict-free Replicated Data Types (CRDTs)

Key Idea: CRDTs allow multiple replicas to perform independent updates and then merge these updates automatically. This ensures that all replicas eventually converge without needing explicit conflict resolution.

Correctness Guarantees: CRDTs provide eventual consistency but not necessarily strong consistency. The state on each replica may be slightly different, but the final converged state will be consistent across all replicas.

Engineering Trade-offs

Simplicity: Implementing CRDTs is generally simpler because they don't require complex transformation logic.
Performance: Operations are local and simple, making them perform well in a distributed environment.
Scalability: Highly scalable but may suffer from increased state complexity over time.
Tombstone Growth

Tombstone Growth: In both OT and CRDTs, tombstones represent states that become obsolete due to conflicts or failed updates. Over time, the amount of tombstone data can grow significantly, especially in systems with high concurrency.

OT: Tombstones are generated when transformations fail to apply correctly, leading to temporary inconsistencies.
CRDT: Tombstones may accumulate as replicas converge and diverge, leading to state fragmentation over time.

#### Closing Summary

In a collaborative editor setting, CRDTs offer a simpler implementation with better performance and scalability. However, they introduce the risk of increased state complexity and tombstone growth. OT provides strong consistency guarantees but at the cost of higher computational overhead and potential latency issues.

#### Common Mistakes/Follow-ups
Overlooking Tombstone Management: Both methods require careful management of tombstones to avoid excessive storage and performance impacts.
Ignoring Long-term Consistency: Ensuring eventual consistency is crucial, especially in systems with high concurrency.
Neglecting Real-world Workloads: Evaluating the system under real-world usage patterns can reveal hidden complexities.

**Comparison:**

| Replica ID | Initial State | Operation Applied (PbOps) | Final State |
|------------|---------------|---------------------------|-------------|
| 1          | A             | [B, C]                     | ABC         |
| 2          | B             | []                         | B           |

**Code:**

```text
# Example of CRDTs implementation (PbOp + Accumulator)
class PbOp:
    def __init__(self, value):
        self.value = value

class AccumulatorCRDT:
    def __init__(self):
        self.ops = []

    def apply(self, op: PbOp):
        self.ops.append(op)

    def merge(self, other_crdt):
        for op in other_crdt.ops:
            if op not in self.ops:
                self.apply(op)
```

**Example input / output:**

| Replica ID | Initial State | Operations Applied (PbOps) | Final State |
|------------|---------------|---------------------------|-------------|
| 1          | version1      | [PbOp("Updated content")]   | version2    |
| 2          | version1      | []                        | version1    |

**Data:**

```json
{
  "document": {
    "version1": "Initial document content",
    "version2": "Updated document content"
  },
  "replicas": [
    {"id": 1, "content": "version1"},
    {"id": 2, "content": "version1"}
  ]
}
```

**Flow:**

1. Applying PbOp('C')
2. No operations
3. Merge with Replica 2
4. Replica 1
5. CRDT State: AC
6. Replica 2
7. CRDT State: B
8. CRDT State: AB

