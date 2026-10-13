---
type: "practice"
difficulty: "Expert"
---

# You're seeing lock contention and deadlocks on a hot B-Tree page under high concurrency. Explain the difference between row locks and page latches, how gap locks in InnoDB REPEATABLE READ contribute, and how you'd reduce it.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ B-tree Concurrency and Latches](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_Concurrency_and_Latches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with contention issues related to B-Tree pages within a database system like MySQL, which uses the InnoDB storage engine. The primary issue is lock contention and deadlocks, particularly around page latches in a high-concurrency environment. To understand and address this problem, it's crucial first to distinguish between row locks (used by InnoDB) and page latches (used by B-Tree pages). Then, we need to explore how gap locks contribute to maintaining transactional consistency under the REPEATABLE READ isolation level. Finally, I will outline strategies for reducing these contention issues.

Row locks are used in InnoDB at the row-level granularity to manage concurrent transactions on individual rows. In contrast, page latches (also known as page locks) protect an entire B-Tree page and prevent multiple transactions from modifying the same page simultaneously, leading to potential lock contention if many transactions access the same page frequently.

InnoDB employs gap locks in REPEATABLE READ isolation level to prevent phantom reads—where a row can appear or disappear between two consecutive queries. However, when combined with shared and exclusive row locks, this can lead to complex locking scenarios, especially under high concurrency, where deadlocks might occur due to the interplay of these different types of locks.

To reduce lock contention and deadlocks in such scenarios, we can implement several strategies:

Index Optimization: Ensure that indexes are well-designed to minimize the number of pages accessed during a query. This can be achieved by creating composite indexes or covering indexes.
Query Refinement: Optimize queries to avoid unnecessary full table scans or page-level locks. For instance, using appropriate WHERE clauses and avoiding SELECT * from large tables.
Concurrency Control Mechanisms: Adjust InnoDB configuration parameters like innodb_lock_wait_timeout and innodb_deadlock_detect, which control how long a transaction will wait before timing out due to lock contention and initiating deadlock detection, respectively.
Partitioning: Partition the table based on a relevant column to reduce the likelihood of multiple transactions accessing the same page.
Asynchronous Operations: Where feasible, use asynchronous operations or background tasks to offload heavy processing that might otherwise block other transactions.

**Comparison:**

| Column      | Type    |
|-------------|---------|
| id          | integer |
| name        | varchar |
| department_id | int     |
| salary      | float   |

**Code:**

```sql
-- Example SQL queries to demonstrate index optimization and query refinement

CREATE INDEX idx_employee_dept ON employees(department_id);

SELECT * FROM employees WHERE department_id = 10 AND salary > 50000;
```

**Example input / output:**

| Input Query | Output Result |
|-------------|---------------|
| SELECT * FROM employees WHERE department_id = 10 AND salary > 50000; | Returns rows with id=1 and id=2 from the 'employees' table where their departments have ids equal to 10, and their salaries are greater than 50000. |

**Data:**

```json
{
  "table": {
    "columns": ["id", "name", "department_id", "salary"],
    "rows": [
      [1, "Alice", 10, 60000],
      [2, "Bob", 10, 55000],
      [3, "Charlie", 20, 70000]
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Transaction Starts
4. Is Index Covering?
5. Use Index to Minimize PageAccess
6. Optimize Query Using WHEREClauses
7. Reduce Lock Contention byIndexing
8. Create Covering Index orOptimize Query
9. Adjust InnoDB ConfigurationParameters
10. Partition Table Based onRelevant Column
11. Set innodb_lock_wait_timeoutand innodb_deadlock_detect
12. Offload Heavy Processing toBackground Tasks
13. Reduce Lock Contentionduring Query Execution
14. Improve Concurrency byOffloading Workloads

