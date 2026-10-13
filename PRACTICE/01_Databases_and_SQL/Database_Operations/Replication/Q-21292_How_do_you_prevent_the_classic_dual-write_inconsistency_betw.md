---
type: "practice"
difficulty: "Expert"
---

# How do you prevent the classic dual-write inconsistency between the primary and downstream stores by treating the replication log as the single ordering authority?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Replication Topologies and Lag](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Replication_Topologies_and_Lag.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address dual-write inconsistencies in a master-slave database replication setup, we need to treat the replication log as the single source of truth. The core idea is to ensure that all writes are first applied to the primary (master) database and then replicated to downstream replicas (slaves). This prevents scenarios where both the primary and a slave might receive conflicting updates simultaneously.

To begin, let's summarize the key aspects: we need to capture every write operation on the master in its replication log. Then, slaves must strictly apply these operations from the log in the same order they were received, ensuring no concurrent writes or out-of-order application. This approach leverages the principle of eventual consistency while maintaining a strict ordering.

**Comparison:**

| Operation          | Master           | Slave 1         | Slave 2         |
|--------------------|------------------|-----------------|-----------------|
| Time               | 09:00 AM         | 09:05 AM        | 09:10 AM        |
| Operation          | INSERT INTO orders (customer_id, product_id, quantity) VALUES (1, 2001, 5); | `Received`       | `Received`      |
| Log Entry          | [log_id_1]       | [log_id_2]      | [log_id_3]      |

**Code:**

```sql
-- Example SQL operation on the master
INSERT INTO orders (customer_id, product_id, quantity) VALUES (1, 2001, 5);
```

**Example input / output:**

```text
- **Input**: A write operation `INSERT INTO orders (customer_id, product_id, quantity) VALUES (1, 2001, 5);` is executed on the master.
- **Output**: The same operation is recorded in the replication log. Then, it is propagated to all slaves for execution.
```

**Flow:**

1. No
2. Primary DB Write Operation
3. Replication Log Entry
4. Is Replication Completed?
5. Send Log to Slaves
6. Slaves Apply Log Entries inOrder

