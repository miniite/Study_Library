---
type: "practice"
difficulty: "Intermediate"
---

# What are the first three things you would check when someone reports a suddenly slow query in production?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Diagnosing Query Regressions](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Diagnosing_Query_Regressions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a slow query reported by a user or monitoring system, the first thing to understand is that there could be multiple underlying issues causing performance degradation. The three initial steps I would take are:

Identify and Understand the Query: Review the problematic SQL statement to grasp its structure and expected behavior.
Check Execution Plan: Analyze how the database executes this query, looking for inefficiencies or unexpected behaviors.
Monitor Database Metrics: Inspect real-time metrics like CPU usage, memory utilization, I/O operations, and lock waits to see if there are broader system issues.

Let’s dive into each of these steps with more detail.

**Code:**

```sql
-- Example slow query identified in production
SELECT a.col1, b.col2
FROM table_a a
JOIN table_b b ON a.id = b.id
WHERE a.date >= '2023-01-01';
```

**Example input / output:**

```

Given the example query:

SQL
Copy
SELECT a.col1, b.col2
FROM table_a a
JOIN table_b b ON a.id = b.id
WHERE a.date >= '2023-01-01';

Initial analysis suggests that despite table_a having an index on id, the lack of indexing in table_b could lead to suboptimal performance.
```

**Data:**

```plaintext
Table A (table_a):
- 1M rows
- Single index on `id` and `date`

Table B (table_b):
- 50K rows
- No indexes, except for a unique constraint on the `id` column
```

