---
type: "practice"
difficulty: "Deep"
---

# A query that filters on the partition key still scans every partition. Walk me through how you would debug why pruning didn't kick in.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Partition Pruning](<../../../../01_Databases_and_SQL/Database_Operations/Partitioning/Partition_Pruning/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a query filters on the partition key and still scans every partition, it suggests that there might be an issue with how the database system is handling partition pruning. This could stem from configuration, indexing, or even a bug in the query optimizer. Our goal is to systematically debug this issue by checking several aspects of the database setup and query execution.

First, let's lay out what we know: the query filters on the partition key, but it still scans all partitions. Partition pruning should ideally limit the scan to only those partitions that could contain relevant data based on the filter conditions. We need to ensure our database is configured correctly for partitioning, verify the query and its execution plan, check for potential bugs or suboptimal configurations in the optimizer, and validate the schema and indexes.

**Comparison:**

```
tablePartitioning {
  header("Sales Table", "Partitions")
  row(0, 2022-01-01, 2022-06-01)
  row(1, 2022-06-01, 2023-01-01)
  row(2, 2023-01-01, 2023-06-01)
  row(3, 2023-06-01, 2024-01-01)
}
```

**Code:**

```sql
-- Example Query that filters on a partition key but still scans all partitions
SELECT * FROM sales WHERE order_date >= '2023-01-01' AND order_date < '2023-06-01';
```

**Example input / output:**

```text
-- Input Query
SELECT * FROM sales WHERE order_date >= '2023-01-01' AND order_date < '2023-06-01';

-- Expected Output (assuming partition pruning works correctly)
[
  {"order_id": 1234, "customer_id": 5678, "order_date": "2023-03-15", ...},
  ...
]

-- Actual Output
[
  {"order_id": 1234, "customer_id": 5678, "order_date": "2023-03-15", ...},
  {"order_id": 9876, "customer_id": 4321, "order_date": "2022-12-31", ...},
  ...
]
```

**Data:**

```json
{
  "table": {
    "name": "sales",
    "partition_key": "order_date"
  },
  "index": [
    {
      "name": "idx_order_date",
      "type": "B-tree",
      "columns": ["order_date"]
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Identify the Problem
6. Is partition pruning disabled?
7. Check database configuration
8. Verify indexing on partitionkey
9. Examine query execution planfor prune optimization
10. Are there any explicit queriesdisabling pruning?
11. Debug optimizer settings andrules
12. Check the query for explicitpartition scanning
13. Validate schema andconstraints affectingpartitioning
14. Review database logs forerrors or warnings duringquery execution

