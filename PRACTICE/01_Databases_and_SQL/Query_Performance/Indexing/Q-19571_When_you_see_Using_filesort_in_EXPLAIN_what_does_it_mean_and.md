---
type: "practice"
difficulty: "Intermediate"
---

# When you see "Using filesort" in EXPLAIN, what does it mean and is it always bad?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN ANALYZE](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Reading_EXPLAIN_ANALYZE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you encounter "Using filesort" in a MySQL query's EXPLAIN output, it indicates that the database needs to perform an additional sort operation on the result set after fetching rows from the table. This can often signify inefficiency or suboptimal indexing, but it’s not always detrimental.

To fully understand this concept and its implications, we need to break down what "Using filesort" means, explore when it might be necessary, and evaluate whether it's a good practice. Let's start by defining the key terms and then walk through an example to illustrate the issue.

**Comparison:**

| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra           |
|----|-------------|-------|-------|--------------|---------|---------|-------|------|----------|-----------------|
| 1  | SIMPLE      | cust  | ALL   | NULL         | NULL    | NULL    | NULL  | 5000 | 2.00     | Using filesort; Using where |

**Code:**

```sql
-- Example query causing filesort
EXPLAIN SELECT * FROM customers WHERE name = 'John' ORDER BY id;
```

**Example input / output:**

```

Input: EXPLAIN SELECT * FROM customers WHERE name = 'John' ORDER BY id;

Output:

PLAINTEXT
Copy
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra           |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
|  1 | SIMPLE      | cust  | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | 5000 |   2.00    | Using filesort; Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
```

**Data:**

```plaintext
Table: customers
Columns:
- id (INT)
- name (VARCHAR)
- email (VARCHAR)
- created_at (DATETIME)
```

**Flow:**

1. Query: SELECT * FROMcustomers WHERE name ='John' ORDER BY id
2. EXPLAIN output
3. Using filesort' and 'Usingwhere
4. Rows fetched are not in sortedorder
5. Additional sort operationrequired

