---
type: "practice"
difficulty: "Deep"
---

# You have a composite index on (a, b, c) — explain the leftmost-prefix rule and which of these predicates it can serve: a=?, b=?, a=? AND c=?.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composite Indexes and Column Order](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Composite_Indexes_and_Column_Order.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database indexing, understanding how an index is used by the query optimizer is crucial to optimizing performance. The leftmost-prefix rule is a key principle that helps in determining which indexes can be used efficiently by the query planner, especially when multiple columns are involved.

The leftmost-prefix rule states that for a composite index on (a, b, c), the database engine will use this index if the conditions start with any of its prefix columns. Let's break down how it applies to each predicate:

a=?: This predicate matches the first column of our composite index. Therefore, the leftmost-prefix rule guarantees that the index can be used efficiently for this query.
b=?: This predicate involves a second column in the index but not the first one. According to the leftmost-prefix rule, an index on (a, b, c) cannot be used when the condition starts with b. Hence, it will not serve this predicate.
a=? AND c=?: While this query matches the start of our composite index and also involves the third column, it does not fully utilize the power of a composite index due to its intermediate column b not being involved in the condition.

Let’s illustrate with an example:

SQL
CREATE TABLE table_name (a INT, b INT, c INT);
CREATE INDEX idx_ab_c ON table_name(a, b, c);

Suppose we run these queries:

Query 1: WHERE a = 5 - The leftmost-prefix rule allows the use of idx_ab_c because it starts with column a.
Query 2: WHERE b = 6 - This query cannot benefit from idx_ab_c as it does not start with column a.
Query 3: WHERE a = 5 AND c = 10 - While this involves the first and third columns, the second column (b) is missing from the condition. The leftmost-prefix rule thus limits its effectiveness for this query.

In conclusion, the leftmost-prefix rule ensures that composite indices are used efficiently by starting with their prefix columns. For idx_ab_c, only a queries can benefit directly as they involve the first column of the index.

**Code:**

```sql
CREATE TABLE table_name (a INT, b INT, c INT);
CREATE INDEX idx_ab_c ON table_name(a, b, c);
```

**Example input / output:**

```text
- **Input 1:** `WHERE a = 5`
  - **Output 1:** The query can use the index `idx_ab_c`.
- **Input 2:** `WHERE b = 6`
  - **Output 2:** The query cannot use the index `idx_ab_c` as it does not start with column `a`.
- **Input 3:** `WHERE a = 5 AND c = 10`
  - **Output 3:** The query can use the index, but its efficiency is limited because of missing conditions on column `b`.
```

