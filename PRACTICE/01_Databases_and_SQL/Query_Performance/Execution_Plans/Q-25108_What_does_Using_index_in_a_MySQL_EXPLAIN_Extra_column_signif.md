---
type: "practice"
difficulty: "Intermediate"
---

# What does 'Using index' in a MySQL EXPLAIN Extra column signify?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Reading Execution Plans](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Reading_Execution_Plans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of MySQL query optimization, the Using index field within an EXPLAIN output indicates that the database is performing a query using only the indexed columns directly without fetching any additional data from the table's body. This can be beneficial in terms of performance since it reduces I/O operations and memory usage.

To understand this concept fully, let's break down what Using index means step by step.

**Code:**

```sql
-- Example query that uses an index
EXPLAIN SELECT name FROM employees WHERE id = 123;
```

**Example input / output:**

```text
For the given query, the `EXPLAIN` output might look like this:

| id | select_type | table       | type   | possible_keys | key     | key_len | ref    | rows  | Extra          |
|----|------------|-------------|--------|---------------|---------|---------|--------|-------|----------------|
| 1  | SIMPLE     | employees   | const  | PRIMARY       | PRIMARY | 4       | const  | 1     | Using index    |

Here, `const` indicates a constant where the column value is known and can be used to locate specific rows quickly. The `Using index` extra field tells us that only the indexed columns are being read.
```

**Flow:**

1. Query Execution Starts
2. Select Type
3. Table
4. Type
5. Possible Keys
6. Key
7. Key Len
8. Ref
9. Rows
10. Using index

