---
type: "practice"
difficulty: "Expert"
---

# Design the multi-region write-conflict-resolution strategy for user-editable data where LWW would silently lose edits — evaluate CRDTs, app-level merge, and single-writer-per-key at organizational scale.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-region Write Paths](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Cross-region_Write_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed system with multiple regions, ensuring consistency of user-editable data while handling write conflicts is critical. The challenge here is to design a strategy that avoids silent loss of edits (like Last-Writer-Wins, LWW) and evaluate CRDTs (Conflict-free Replicated Data Types), app-level merge strategies, and single-writer-per-key approaches at an organizational scale.

#### Triaging the Question

The core question involves choosing an appropriate conflict resolution mechanism for a multi-region distributed system. The interviewer is probing your understanding of various consistency models and their trade-offs in a high-scale environment. Key assumptions include:

High write volume across regions.
Latency between regions may vary significantly.
Data integrity and avoiding silent data loss are paramount.

#### First Principles Solution
CRDTs (Conflict-free Replicated Data Types): CRDTs inherently handle conflicts without requiring coordination or communication, making them ideal for distributed systems. However, they can be complex to implement and require careful design to ensure data consistency.
App-Level Merge: This involves defining a merge strategy within the application code. While it provides more control over how writes are merged, it requires robust implementation and maintenance efforts.
Single-Writer-Per-Key (SWPK): Ensuring only one writer can modify a key at any given time across all regions. This approach guarantees consistency but may introduce bottlenecks.
Evaluating CRDTs

CRDTs like Vector Clocks or Merkle Trees are designed to handle conflicts without coordination. For example, in a Vector Clock-based system:

Each data object maintains a vector of (node, timestamp) pairs.
When conflicting writes occur, the latest write wins based on vector clock comparisons.

#### Pros

Decentralized conflict resolution.
No need for centralized coordination.

#### Cons

Complex to implement and debug.
May result in larger data size due to metadata overhead.
Evaluating App-Level Merge

App-level merge involves defining custom logic within the application. For instance, merging a list might involve concatenation or taking the union of both lists.

#### Pros

Flexibility in defining complex merge strategies.
Can be optimized based on specific business requirements.

#### Cons

Requires significant development effort to implement and test merges.
Maintenance overhead for ensuring correctness across all use cases.
Evaluating Single-Writer-Per-Key

SWPK ensures consistency by allowing only one writer per key. This can be enforced using distributed locks or consensus protocols like Raft.

#### Pros

Guaranteed consistency with no conflict resolution needed.
Simplifies application logic since merge strategies are not required.

#### Cons

Can introduce bottlenecks and single points of failure.
Requires coordination mechanisms that increase complexity.
Organizational Scale Considerations

At an organizational scale, the solution must handle numerous regions and high write volumes efficiently. Key considerations include:

Scalability: Ensuring the chosen strategy scales well with increasing numbers of regions and users.
Fault Tolerance: Handling network partitions and ensuring data consistency during partial outages.
Cost: Evaluating the cost implications, including operational complexity and infrastructure costs.

#### Closing Summary

In conclusion, CRDTs provide a decentralized approach that avoids complex merge logic but requires careful implementation. App-level merge offers flexibility but demands robust coding practices. SWPK guarantees consistency but introduces potential bottlenecks. The choice depends on specific organizational requirements and trade-offs between complexity, performance, and cost.

#### Common Mistakes
Overlooking the importance of data integrity in favor of simplicity.
Failing to consider edge cases and failure modes.
Ignoring operational costs associated with chosen strategies.

**Comparison:**

```
Consistency Model	Pros	Cons
CRDTs	Decentralized, No Coordination	Complex Implementation, Metadata Overhead
App-Level Merge	Flexible Merging Logic	High Development & Maintenance Effort
SWPK	Guaranteed Consistency	Potential Bottlenecks, Requires Coordination
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[CRDTs] --> B(Vector Clocks)
    A --> C[Merkle Trees]
    D[App-Level Merge] --> E[List Concatenation]
    D --> F[Custom Merging Logic]
    G[Single-Writer-Per-Key] --> H[Distributed Locking]
```

**Example input / output:**

```text
Input (User edits data in region-A):
- User updates profile with new email.

Output (Conflict resolution mechanism decides how to handle the update in region-B):
- CRDT: Vector Clock resolves conflict based on timestamps.
- App-Level Merge: Custom merge logic ensures no duplicate emails.
- SWPK: Single-writer-per-key lock is acquired, ensuring atomic updates.
```

**Data:**

```yaml
# Example CRDT Data Structure (Vector Clock)
user_data:
  id: "123"
  vector_clocks:
    - {node: "region-A", timestamp: 1577836800}
    - {node: "region-B", timestamp: 1577836900}

# Example App-Level Merge Logic
merge_lists:
  function(list_a, list_b) {
      return [...list_a, ...list_b.filter(item => !list_a.includes(item))];
  }
```

**Flow:**

1. User Edit in Region-A
2. CRDT Conflict Resolution
3. Conflict Resolved with VectorClock
4. User Edit in Region-B
5. App-Level Merge Logic
6. Merged Lists Update
7. SWPK Lock Acquired
8. Synchronized Write to AllRegions

