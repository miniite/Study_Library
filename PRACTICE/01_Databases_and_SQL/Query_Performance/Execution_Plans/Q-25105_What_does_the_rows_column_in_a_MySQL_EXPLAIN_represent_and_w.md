---
type: "practice"
difficulty: "Intermediate"
---

# What does the 'rows' column in a MySQL EXPLAIN represent and why is it only an estimate?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Reading Execution Plans](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Reading_Execution_Plans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing the rows column in a MySQL EXPLAIN, it's important to understand that this value provides an estimate of the number of rows the query optimizer believes will be returned by a particular step of the query execution plan. This is only an estimate because the optimizer uses statistical information from indexes, table statistics, and other factors to make educated guesses about how many rows need to be processed at each stage.

The rows column helps in identifying potential bottlenecks in the query's execution plan and can guide us in understanding where improvements might be necessary. However, it’s crucial not to treat this value as a hard count, as it can vary based on the data distribution and other conditions.

**Code:**

```sql
-- Example query that will produce an EXPLAIN output with 'rows' column
EXPLAIN SELECT * FROM employees WHERE salary > 1000;
```

**Example input / output:**

```text
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | employees | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 5000 |   2.37 % | Using where |
+----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------+
```

