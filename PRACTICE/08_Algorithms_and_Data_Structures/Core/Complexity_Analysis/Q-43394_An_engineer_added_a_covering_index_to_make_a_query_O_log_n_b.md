---
type: "practice"
difficulty: "Deep"
---

# An engineer added a covering index to make a query O(log n) but writes regressed; walk through the read-vs-write complexity trade-off and how you'd decide if the index is worth keeping.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sorting Cost in Services](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Cost_in_Services.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand whether an added covering index on a database table made a query O(log n) but caused performance regression, we need to carefully analyze both read and write complexities of operations in that index and decide based on business requirements and workload characteristics. The core idea is that while indexes can speed up reads by reducing the number of I/Os required, they can slow down writes due to the overhead of maintaining them.

The question probes your understanding of:

How indexes impact read performance.
How indexes affect write operations.

#### Trade-offs
between read and write performance.
Criteria for deciding whether an index is beneficial.

Let's break this down from first principles:

Read Performance

Adding a covering index means that all the columns required by the query are included in the index itself. This eliminates the need to fetch data from the actual table, significantly reducing I/O operations and often speeding up queries from O(n) to O(log n). However, this comes with overhead:

Disk Space: More space is used for storing the index.
Maintenance Overhead: Indexes require regular maintenance (e.g., updates, deletes, rebalancing), which can impact performance.
Write Performance

Indexes themselves can slow down write operations because each insert, update, or delete operation now needs to be reflected in both the main table and the index. This doubling of work can degrade performance:

Inserts: Need to update the index.
Updates: Must update the value in the index along with the main table.
Deletes: Need to remove entries from the index.
Trade-off Decision

To decide if keeping the index is worthwhile, we need to consider several factors:

Query Frequency and Importance: Are these queries critical? How often are they executed?
Write Operations: How many writes occur compared to reads? Are there many updates/inserts/deletes happening on this table?
Table Size: Larger tables might benefit more from index optimization.
Business Impact: What is the business impact of slow query performance versus the cost of maintaining indexes?

#### Example Scenario

Suppose we have a user table with millions of rows and frequent queries on the email column, which were previously O(n) but are now O(log n). However, there are also frequent updates to the status field. If the write operations significantly outweigh read operations (e.g., 10 times more), keeping the index might not be justified.

Detailed Analysis

Let's assume:

Read Operations: 95% of total queries.
Write Operations: 5% of total queries.

If maintaining the index incurs a 2x write overhead but provides only 3x read speedup, it’s likely that removing the index would improve overall system performance.

#### Conclusion

In conclusion, deciding whether to keep an index involves balancing its benefits and costs. You should:

Measure current read/write performance without the index.
Simulate or measure the impact of adding/keeping the index.

#### Cons
ider business priorities and SLAs.
Monitor over time to ensure the decision aligns with evolving workloads.

#### Common mistakes
include:

Not considering all types of operations (just reads).
Overestimating write overhead without empirical data.
Ignoring business context and user experience.

**Comparison:**

| Operation | Current Performance | With Index | Without Index |
|-----------|---------------------|------------|---------------|
| Read      | 2 sec               | 0.5 sec    | 1 sec         |
| Write     | 3 sec               | 6 sec      | 4 sec         |

**Code:**

```python
# Example Python code for simulating read/write performance

def simulate_performance(reads, writes):
    # Simulate reads
    start = time.time()
    for _ in range(reads):
        pass  # Placeholder for actual read logic
    read_time = time.time() - start

    # Simulate writes
    start = time.time()
    for _ in range(writes):
        pass  # Placeholder for actual write logic
    write_time = time.time() - start

    return read_time, write_time

reads, writes = simulate_performance(100000, 5000)
print(f"Read Time: {reads} sec, Write Time: {writes} sec")
```

**Example input / output:**

```text
Current Read/Write Performance:
- Reads: 2 sec
- Writes: 3 sec

With Index:
- Reads: 0.5 sec (3x speedup)
- Writes: 6 sec (2x overhead)

Without Index:
- Reads: 1 sec
- Writes: 4 sec
```

**Data:**

```json
{
  "table": {
    "name": "user",
    "columns": ["id", "email", "status"],
    "index": {
      "type": "covering",
      "on": ["email"]
    },
    "rows": 1000000
  }
}
```

**Flow:**

1. Yes
2. No
3. Measure Current Performance
4. Is Index Beneficial?
5. Keep Index
6. Remove Index
7. Maintain and Monitor
8. Optimize Writes

