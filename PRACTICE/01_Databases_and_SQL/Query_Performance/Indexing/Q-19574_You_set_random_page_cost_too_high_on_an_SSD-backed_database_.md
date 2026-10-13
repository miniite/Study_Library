---
type: "practice"
difficulty: "Expert"
---

# You set random_page_cost too high on an SSD-backed database. What symptom would you expect in query plans?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Offset Pagination Pitfalls](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Offset_Pagination_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting random_page_cost too high on an SSD-backed database, we can anticipate several adverse effects on query performance and query optimization. At a backend architect/principal level, understanding these impacts is crucial to make informed decisions about indexing strategies and query optimization.

The primary issue stems from the assumption that SSDs are much faster than traditional spinning disks. random_page_cost represents the estimated cost of an I/O operation (read or write) on disk. Setting this parameter too high leads the query planner to overestimate the cost of reading data from SSD, leading it to choose suboptimal plans.

To diagnose and mitigate these issues, we need to understand what symptoms to look for in query execution plans and how to adjust our database configuration accordingly.

**Comparison:**

```
Query Plan with High random_page_cost	Expected Optimal Plan
Full Table Scan (Cost=310)	Index Scan (Cost=5)
```

**Code:**

```

-- Example SQL query SELECT * FROM sales WHERE date > '2023-01-01';

CODE
Copy

```yaml
# PostgreSQL configuration snippet
random_page_cost: 4.0
```

**Example input / output:**

```text
When `random_page_cost` is set to a high value, like 4.0, the query planner might choose less efficient plans over more optimal ones. For example:

- A simple range scan that could benefit from an index on `date` would be deemed too expensive by the query planner and thus ignored.
- The planner may opt for full table scans or use sequential scans even when an index exists.

This results in slower performance and suboptimal resource utilization, which can impact the overall database throughput and user experience.
```

**Flow:**

1. High random_page_cost
2. Query Planner OverestimatesI/O Costs
3. Chooses Less Efficient Plans
4. Slower Performance, Full TableScans

