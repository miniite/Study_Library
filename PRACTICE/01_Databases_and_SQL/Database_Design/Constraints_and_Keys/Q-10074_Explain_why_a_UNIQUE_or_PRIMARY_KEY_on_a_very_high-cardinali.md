---
type: "practice"
difficulty: "Deep"
---

# Explain why a UNIQUE or PRIMARY KEY on a very high-cardinality wide column can bloat the index and hurt insert throughput. How would you narrow it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Primary and Unique Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database, understanding how constraints like unique keys and primary keys affect performance is crucial. A high-cardinality wide column in such constraints can lead to significant bloat in the index, which can negatively impact insert throughput. To understand this, let's break down the problem from first principles.

**Comparison:**

| Column | Type   |
|--------|--------|
| id     | BIGINT |
| column1| TEXT   |
| column2| TEXT   |

**Code:**

```sql
-- Example of a table with a wide unique key constraint
CREATE TABLE large_table (
    id BIGINT PRIMARY KEY,
    column1 TEXT,
    column2 TEXT,
    ... -- Many more columns
);
```

**Flow:**

1. Yes
2. No
3. Create Table with Wide UniqueKey
4. Cardinality High?
5. Index Bloat Occurs
6. Insert Throughput Decreases
7. Insert Performance RemainsStable

