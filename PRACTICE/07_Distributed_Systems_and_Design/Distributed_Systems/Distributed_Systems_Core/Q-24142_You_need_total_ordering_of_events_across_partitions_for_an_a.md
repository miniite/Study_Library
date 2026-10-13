---
type: "practice"
difficulty: "Deep"
---

# You need total ordering of events across partitions for an audit table but also throughput. Discuss the tension and options: a single partition, a sequencer service, logical timestamps in the row, or reordering at read time in SQL.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Ordering and Partition Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Ordering_and_Partition_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve total ordering of events across partitions while maintaining high throughput in an audit table, we need to carefully consider the trade-offs and options available. The core tension lies between ensuring consistent, ordered data and maximizing read/write performance. Let's explore each option in detail.

#### First-principles solution

Single Partition: This approach simplifies the ordering of events as all writes go to a single location. However, it severely limits throughput since every write must go through this partition.

Sequencer Service: A dedicated service can assign unique sequence numbers to each event, ensuring total ordering. This approach maintains high throughput but requires additional coordination and potential overhead.

Logical Timestamps in the Row: Adding a logical timestamp directly to each row allows for natural ordering without needing external sequencing. However, this can lead to complexities in managing timestamps across partitions.

Reordering at Read Time: Post-processing data during read operations can ensure ordered results but introduces complexity and potential performance overhead.

Each option has its pros and cons, which we'll evaluate step-by-step.

**Comparison:**

| Option       | Pros                          | Cons                           |
|--------------|-------------------------------|--------------------------------|
| Single Partition | Simple ordering, no sequencing service  | Low throughput, single point of failure |
| Sequencer Service    | High throughput, easy to scale     | Additional complexity, potential coordination overhead   |
| Logical Timestamps | No external sequences required, natural order      | Complex timestamp management, may introduce write contention  |
| Reordering at Read Time | Post-processing ensures order, avoids sequence service  | Can be performance-intensive, adds complexity    |

**Code:**

```sql
-- Example SQL schema for the audit table with logical timestamps
CREATE TABLE audit (
    id INT PRIMARY KEY,
    event_type VARCHAR(255),
    payload JSON,
    timestamp BIGINT -- logical timestamp in milliseconds
);
```

**Example input / output:**

```text
-- Example of reordering events at read time using SQL window functions
SELECT id, event_type, payload, timestamp
FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY timestamp) AS row_num
    FROM audit
) t
ORDER BY row_num;
```

**Data:**

```json
{
  "events": [
    {"id": 1, "event_type": "login", "payload": {}, "timestamp": 1672539000},
    {"id": 2, "event_type": "logout", "payload": {}, "timestamp": 1672540000},
    {"id": 3, "event_type": "login", "payload": {}, "timestamp": 1672541000}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Single Partition
4. Write to Single Location
5. Event Sequence?
6. Post-Processing at Read Time
7. Use Sequencer Service
8. Logical Timestamps in Row
9. Assign Timestamp on Write
10. Reordering at Read Time
11. Read and Reorder Events

