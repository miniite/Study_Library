---
type: "practice"
difficulty: "Deep"
---

# You need a distributed sequence/ticket service that never issues a duplicate or a backwards ID at millions/sec and survives node loss. Compare a consensus-backed single allocator, batched range leases per node, and hybrid-logical-clock IDs on availability, ordering strength, and coordination cost.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed ID Generation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_ID_Generation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a distributed sequence/ticket service that handles millions of requests per second while ensuring no duplicates or backward IDs, we need to compare three approaches: a consensus-backed single allocator, batched range leases per node, and hybrid-logical-clock (HLC) IDs. We'll evaluate each on availability, ordering strength, and coordination cost.

**Comparison:**

| Approach                  | Consensus Backed Single Allocator | Batched Range Leases per Node | Hybrid Logical Clocks    |
|---------------------------|----------------------------------|-------------------------------|--------------------------|
| **Ordering Strength**     | Strong                           | Weak to Moderate               | Strong, with backoff     |
| **Availability**          | Moderate to High                 | Moderate to High               | High                     |
| **Coordination Cost**     | High                             | Low to Moderate                | Moderate                 |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Consensus Backed Single Allocator] --> B1(Strong Ordering) --> C1(Availability);
    A --> D1(Coordination Cost) --> E1(High);
    A --> F1(Duplicate/Backward ID Prevention) --> G1(Strong);

    B2[Batched Range Leases per Node] --> H1(Ordering) --> I1(Weak to Moderate);
    B2 --> J1(Availability) --> K1(Moderate to High);
    B2 --> L1(Coordination Cost) --> M1(Low to Moderate);

    B3[HLC IDs] --> N1(Ordering) --> O1(Strong, with Backoff);
    B3 --> P1(Availability) --> Q1(High);
    B3 --> R1(Duplicate/Backward ID Prevention) --> S1(Moderate);
```

**Example input / output:**

| Request Sequence | Consensus Backed Single Allocator | Batched Range Leases per Node | Hybrid Logical Clocks |
|------------------|---------------------------------|-------------------------------|-----------------------|
| 1                | [0, High Coordination]           | [1, Low Coordination]          | [1, Low Coordination]  |
| 2                | [1, High Coordination]           | [2, Low Coordination]          | [2, Low Coordination]  |
| ...              | ...                             | ...                           | ...                   |

**Data:**

```json
{
  "ConsensusBacked": {
    "OrderingStrength": "Strong",
    "Availability": "Moderate to High",
    "CoordinationCost": "High"
  },
  "BatchedRangeLeases": {
    "OrderingStrength": "Weak to Moderate",
    "Availability": "Moderate to High",
    "CoordinationCost": "Low to Moderate"
  },
  "HybridLogicalClocks": {
    "OrderingStrength": "Strong, with Backoff",
    "Availability": "High",
    "DuplicateBackwardIDPrevention": "Moderate"
  }
}
```

**Flow:**

1. Hybrid Logical Clocks
2. Batched Range Leases per Node
3. Consensus Backed Single Allocator
4. Node
5. Request Sequence
6. [0, High Coordination]
7. Batched Range Leases
8. [1, Low Coordination]
9. HLC IDs
10. [2, Low Coordination]
11. High Coordination Cost
12. Low Coordination Cost
13. Moderate Coordination Cost

