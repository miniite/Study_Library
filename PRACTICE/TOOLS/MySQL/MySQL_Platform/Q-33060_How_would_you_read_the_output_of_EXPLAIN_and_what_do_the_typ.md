---
type: "practice"
difficulty: "Intermediate"
---

# How would you read the output of EXPLAIN and what do the type, key, rows, and Extra columns tell you?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how to interpret the EXPLAIN command in MySQL, we need to break down its output into several key fields: type, key, rows, and Extra. This explanation will help you make informed decisions about query optimization based on this information.

#### Triage

The EXPLAIN command provides a detailed execution plan for SQL queries. It is used to identify performance bottlenecks and understand how MySQL executes the query. The fields type, key, rows, and Extra are particularly crucial in diagnosing potential issues with your query's performance.

#### First-Principles Solution

Type: This column tells you about the type of join used by MySQL to retrieve data from the table. Common types include:

system: The row comes directly from a system variable.

#### cons
t: The row is read with a const key lookup, meaning it uses an index or primary key for the look-up.
eq_ref: A unique index scan that retrieves one row per combination of the join columns.
ref: An index range scan on a non-unique index, returning multiple rows if several have the same prefix as in the query.

Key: This field indicates which index MySQL used for the query. It can be:

null if no index was used (table scan).
The name of the key (index) being used.
_system, _const, and so on, depending on the context.

Rows: This gives an estimate of the number of rows that MySQL expects to read from the table in order to satisfy the query. Lower values are generally better since fewer rows means faster execution.

Extra: This field contains additional information about the execution plan. Common entries include:

Using where: Indicates that a WHERE clause was used with a non-indexed condition.
Using index: Suggests that only an index scan is being performed, which can be less efficient than using the full table.
Using temporary, Using filesort: These suggest that MySQL needs to create a temporary table or perform additional sorting to complete the query, which can slow down execution.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes
Overlooking the Extra field and its implications for performance. For instance, using an index but still seeing Using where might indicate that a condition is not being used efficiently.

#### Follow-Ups
You can further investigate by looking at other fields like filtered, which shows the percentage of rows that match the WHERE clause based on the index, and full_scan, which indicates if MySQL performed a full table scan.

#### Example
SQL
EXPLAIN SELECT * FROM orders WHERE order_id = 123;

Output:

CODE
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | orders | NULL       | const| PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+---------+---------+-------+------+----------+-------+
Type: const indicates that MySQL uses a constant value (in this case, the primary key) to directly retrieve the row.
Key: PRIMARY shows that the index used is the primary key.
Rows: 1, which means only one row was read.
Extra: NULL, indicating no additional operations were required.

By understanding these fields, you can refine your queries and optimize performance based on how MySQL executes them.

**Code:**

```sql
EXPLAIN SELECT * FROM orders WHERE order_id = 123;
```

