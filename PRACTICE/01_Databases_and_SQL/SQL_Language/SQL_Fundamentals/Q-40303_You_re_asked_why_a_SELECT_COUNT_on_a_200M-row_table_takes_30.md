---
type: "practice"
difficulty: "Deep"
---

# You're asked why a SELECT COUNT(*) on a 200M-row table takes 30 seconds. Explain why counts are expensive in MVCC databases and the trade-offs of exact vs. estimated vs. maintained counters.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ MVCC Bloat and Vacuum](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/MVCC_Bloat_and_Vacuum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a multi-version concurrency control (MVCC) database like PostgreSQL, performing a SELECT COUNT(*) on a large table with 200 million rows can take 30 seconds due to the overhead of scanning and locking a significant portion of the data. The core issue lies in how MVCC handles concurrent transactions by maintaining multiple versions of tuples. This leads us to explore the trade-offs between exact counts, estimated counts, and maintained counters.

#### Triage
Understanding the Question

The question is probing your understanding of:

How MVCC databases handle row-level locking.
The overhead involved in performing a SELECT COUNT(*) operation.

#### Trade-offs
between accuracy, performance, and resource usage for different methods of counting rows.

#### First-Principles Solution
Why Counts are Expensive

In an MVCC database:

Concurrent Transactions: Each transaction sees a snapshot of the database at its start time. To maintain consistency, the database stores multiple versions of each row.
Row Versioning: For every write operation (insert, update, delete), the database creates new versions of the rows, which can lead to a large number of outdated or partially visible data versions.

When you execute SELECT COUNT(*), the database must:

Traverse all the versioned rows in the table.
Check each row's visibility to current transactions (i.e., whether it is committed and not deleted).
Count only the visible rows, discarding those that are invisible due to concurrent modifications.

This process can be very slow for large tables like the one mentioned, as it involves:

Scanning the entire table: Even if most of the rows have been updated recently.
Checking visibility status of each row, which requires additional metadata lookups and comparisons.

#### Trade-offs
Between Exact vs. Estimated vs. Maintained Counters

Exact Count (Current SELECT COUNT(*)):

#### Pros
Provides 100% accuracy at the cost of performance.

#### Cons
High overhead due to scanning all rows and checking their visibility, leading to longer execution times.

Estimated Count:

#### Pros
Faster as it uses sampling or approximation techniques.

#### Cons
Lower accuracy; may not reflect real-time data changes accurately.

Maintained Counter (Incremental Count):

#### Pros
Maintains an up-to-date count by incrementing a single counter during insertions, deletions, and updates.

#### Cons
Additional complexity in maintaining the counter, potential for race conditions if not implemented carefully.
Detailed Explanation
Exact Counts
In MVCC databases like PostgreSQL:
SQL
SELECT COUNT(*) FROM large_table;
This query requires checking the visibility of each row. The database must acquire locks or use other mechanisms to ensure data consistency during the count operation, which can be resource-intensive.
Estimated Counts
For example, using a sampling technique where only a fraction of rows are counted:
SQL
SELECT COUNT(*) AS estimated_count FROM large_table WHERE random() < 0.1;
This approach significantly reduces the number of rows to check but introduces uncertainty in accuracy.
Maintained Counters
A maintained counter involves incrementing a single row each time an insert, delete, or update occurs:
SQL
UPDATE counter SET count = count + 1 WHERE table_name = 'large_table';
This method is efficient for read-heavy workloads but can introduce complexity and potential race conditions.

#### Closing Summary

In summary, SELECT COUNT(*) on a large MVCC database like PostgreSQL is expensive due to the overhead of scanning all rows and checking their visibility. The trade-offs between exact counts, estimated counts, and maintained counters involve balancing accuracy with performance and resource usage. Exact counts are accurate but slow, while maintained counters provide better performance but may not always reflect real-time data.

#### Common Mistakes/Follow-Ups
Mistake 1: Assuming SELECT COUNT(*) is efficient without considering the overhead of MVCC.

#### Follow-Up
1: Discussing alternative approaches like estimated counts or using statistics tools provided by the database.
Mistake 2: Overlooking the importance of maintaining counters in write-heavy environments.

#### Follow-Up
2: Exploring advanced techniques like approximate query processing.

**Comparison:**

| Row ID | Data     |
|--------|----------|
| 1      | value1   |
| 2      | value2   |
| ...    | ...      |
| 200000000 | valueN   |
| Counter Table |
|---------------|
| Name          | Value         |
| large_table   | 150000000     |

**Code:**

```sql
-- Example exact count
SELECT COUNT(*) FROM large_table;

-- Example estimated count with sampling
SELECT COUNT(*) AS estimated_count FROM large_table WHERE random() < 0.1;

-- Maintained counter update
UPDATE counter SET count = count + 1 WHERE table_name = 'large_table';
```

**Example input / output:**

```text
Input:
SELECT COUNT(*) FROM large_table;

Output:
Execution time: 30 seconds

Input:
SELECT COUNT(*) AS estimated_count FROM large_table WHERE random() < 0.1;

Output:
estimated_count: 25000 (approximate)

Input:
UPDATE counter SET count = count + 1 WHERE table_name = 'large_table';

Output:
Count updated to 150000000
```

**Data:**

```json
{
  "table": {
    "name": "large_table",
    "rows": 200000000,
    "columns": ["id", "data"],
    "constraints": []
  },
  "counter": {
    "name": "count",
    "value": 150000000
  }
}
```

**Flow:**

1. Yes
2. No
3. Sampling
4. Maintained
5. Start
6. Is table large?
7. Perform exact count
8. Execution time: 30 seconds
9. Use sampling or maintainedcounter?
10. Sample rows, getestimated_count
11. Update counter on eachchange
12. Consider alternatives for betterperformance

