---
type: "practice"
difficulty: "Intermediate"
---

# In a MySQL EXPLAIN what does the 'type' column (e.g. ALL/index/ref/const) tell you about access method?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Reading Execution Plans](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Reading_Execution_Plans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The type column in an EXPLAIN plan indicates the type of join or index used by MySQL to retrieve data, which is crucial for understanding and optimizing query performance. At a high level, it tells you how efficiently your query is accessing data from the database.

#### Triage

The question asks us to explain what each value in the type column means in an EXPLAIN plan output. The key idea here is that different values represent different access methods used by MySQL, and some are more efficient than others.

ALL: Indicates a full table scan.
index: Uses an index, but not necessarily the most optimal one.
ref: Uses an index with a single column.

#### cons
t: The query has fewer than 10 rows to return from a single row in another table, often due to a WHERE clause with unique values.

#### First Principles Solution

Let’s break down each type:

ALL (Full Table Scan)

A type value of ALL means that MySQL is performing a full table scan. This is the least efficient way to retrieve data because it requires checking every row in the table, which can be very slow for large datasets.

#### Example

SQL
EXPLAIN SELECT * FROM customers;

Output:

CODE
+----+-------------+------------+------+---------------+------+---------+-------+---------+--------------------------+
| id | select_type | table      | type | possible_keys | key  | key_len | ref   | rows    | Extra                    |
+----+-------------+------------+------+---------------+------+---------+-------+---------+--------------------------+
|  1 | SIMPLE      | customers  | ALL  | NULL          | NULL | NULL    | NULL  | 1000000 | Using where; Using filesort |
+----+-------------+------------+------+---------------+------+---------+-------+---------+--------------------------+
Index

The type value of index means that MySQL is using an index but not necessarily the most optimal one. This can still be slow if multiple columns are used in the WHERE clause, as each column requires a separate index lookup.

#### Example

SQL
EXPLAIN SELECT * FROM customers WHERE first_name = 'John' AND last_name = 'Doe';

Output:

CODE
+----+-------------+------------+-------+---------------+---------+---------+--------------------------+------+--------------------------------------+
| id | select_type | table      | type  | possible_keys | key     | key_len | ref                      | rows | Extra                                |
+----+-------------+------------+-------+---------------+---------+---------+--------------------------+------+--------------------------------------+
|  1 | SIMPLE      | customers  | index | NULL          | idx_name | 508     | const                    | 10   | Using where; Using filesort         |
+----+-------------+------------+-------+---------------+---------+---------+--------------------------+------+--------------------------------------+
Ref

The type value of ref indicates that MySQL is using an index but with a single column. This can be efficient if the query filters by unique values, as it only needs to check one row per unique key.

#### Example

SQL
EXPLAIN SELECT * FROM customers WHERE customer_id IN (1, 2, 3);

Output:

CODE
+----+-------------+------------+-------+---------------+---------+---------+--------------------------+------+--------------------------------------+
| id | select_type | table      | type  | possible_keys | key     | key_len | ref                      | rows | Extra                                |
+----+-------------+------------+-------+---------------+---------+---------+--------------------------+------+--------------------------------------+
|  1 | SIMPLE      | customers  | ref   | idx_customer_id | idx_customer_id | 4       | const                    | 3    | Using where                          |
+----+-------------+------------+-------+---------------+---------+---------+--------------------------+------+--------------------------------------+

#### Cons
t

The type value of const indicates that MySQL is using a constant expression to retrieve data, often due to a WHERE clause with unique values. This is very efficient because it only needs to check one row.

#### Example

SQL
EXPLAIN SELECT * FROM customers WHERE customer_id = 1;

Output:

CODE
+----+-------------+------------+------+---------------+---------+---------+-------+------+--------------------------------------+
| id | select_type | table      | type | possible_keys | key    | key_len | ref   | rows | Extra                                |
+----+-------------+------------+------+---------------+---------+---------+-------+------+--------------------------------------+
|  1 | SIMPLE      | customers  | const| PRIMARY       | PRIMARY | 4       | const | 1    |                                       |
+----+-------------+------------+------+---------------+---------+---------+-------+------+--------------------------------------+

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the type column in an EXPLAIN plan gives us insight into how MySQL is accessing data. Full table scans (ALL) should be avoided if possible. Indexes (index, ref, and const) are generally more efficient but can vary depending on the query’s complexity.

#### Common mistakes
include:

Ignoring full table scans when optimizing queries.
Not understanding the difference between using an index with multiple columns versus a single-column index.
Failing to realize that const is highly optimized for queries with unique values.

#### Follow-ups
might include discussing how to force or change indexing strategies, or explaining the trade-offs between different types of access methods.

**Comparison:**

| id | first_name | last_name | customer_id |
|----|-----------|----------|------------|
| 1  | John      | Doe      | 1001       |
| 2  | Jane      | Smith    | 1002       |
| ...| ...       | ...      | ...        |

**Code:**

```sql
-- Example SQL Queries
SELECT * FROM customers;
EXPLAIN SELECT * FROM customers WHERE first_name = 'John' AND last_name = 'Doe';
EXPLAIN SELECT * FROM customers WHERE customer_id IN (1, 2, 3);
EXPLAIN SELECT * FROM customers WHERE customer_id = 1;
```

**Example input / output:**

```text
Input:
EXPLAIN SELECT * FROM customers WHERE first_name = 'John' AND last_name = 'Doe';

Output:
+----+-------------+------------+-------+---------------+---------+---------+--------------------------+------+--------------------------------------+
| id | select_type | table      | type  | possible_keys | key     | key_len | ref                      | rows | Extra                                |
+----+-------------+------------+-------+---------------+---------+---------+--------------------------+------+--------------------------------------+
|  1 | SIMPLE      | customers  | index | NULL          | idx_name | 508     | const                    | 10   | Using where; Using filesort         |
+----+-------------+------------+-------+---------------+---------+---------+--------------------------+------+--------------------------------------+
```

**Data:**

```sql
-- Sample Table Structure for customers
CREATE TABLE customers (
    id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    customer_id INT UNIQUE
);
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Yes
6. No
7. Yes
8. No
9. Query
10. Full Table Scan (ALL)?
11. Index Usage?
12. Ref Access Method?
13. Const Access Method
14. Single Column Index?
15. Optimized Query
16. Multiple Columns Indexed?
17. More Efficient Query
18. Full Table Scan

