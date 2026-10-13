---
type: "practice"
difficulty: "Expert"
---

# A team reports that adding a region to CockroachDB unexpectedly increased write latency for existing data; explain what happened with survival goals and replica placement, and how you'd have prevented the surprise.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ NewSQL Evaluation and Resilience](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/NewSQL_Evaluation_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a team reported an increase in write latency after adding a new region to CockroachDB, it's important to understand that this issue likely stems from the way CockroachDB handles data distribution and replica placement strategies. In a NewSQL database like CockroachDB, ensuring high availability and consistency is crucial while balancing network latency and write performance.

CockroachDB uses a distributed key-value architecture with a primary replica per partition (shard). Each node can be part of multiple regions, which are defined by the user to map geographically. The system employs a leader election algorithm to decide which node will handle writes for each shard. When a region is added, it could potentially disrupt existing data distribution, leading to increased write latency if not handled properly.

First, let's break down what happened in this scenario and why it might have occurred.

**Code:**

```
MERMAID
Copy
graph TD;
    A[New Region Added] --> B1[Replica Placement Changed];
    B1 --> C1[Write Latency Increased];
```

**Example input / output:**

```text
**Initial Setup:**
- Regions: US-East (R1) and EU-West (R2)
- Shards: [Shard 1, Shard 2]

**After Adding a New Region (R3):**
- Replicas for each shard are reassigned based on the new region layout.
- Write operations to existing data may now involve nodes in R3, increasing network latency.

To prevent such surprises, it's essential to have a detailed understanding of how CockroachDB handles regional changes and replica placement.
```

**Data:**

```json
{
  "regions": [
    {"id": 1, "location": "US-East"},
    {"id": 2, "location": "EU-West"}
  ],
  "shards": [
    {
      "id": 1,
      "replicas": ["node-1", "node-2"],
      "currentRegion": 1
    },
    {
      "id": 2,
      "replicas": ["node-3", "node-4"],
      "currentRegion": 1
    }
  ]
}
```

