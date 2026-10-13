---
type: "practice"
difficulty: "Deep"
---

# How would you use EXPLAIN to diagnose why a query is slow?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why a MySQL query is slow, using EXPLAIN is an essential tool that provides deep insights into how the database executes your query. This technique allows us to see the execution plan of the query and identify potential bottlenecks or inefficiencies in the query's structure and indexing strategy.

First, let’s break down what EXPLAIN probes:

Query Execution Plan: It shows the steps taken by MySQL to retrieve data.
Performance Metrics: Such as the number of rows checked, estimated row count, and whether an index is used or not.

Now, I'll walk you through a step-by-step process using first principles to diagnose slow queries effectively.

Step 1: Understand the Query

#### Cons
ider the following SQL query:

SQL
SELECT * FROM orders WHERE order_date > '2023-01-01';
Step 2: Run EXPLAIN on the Query

Use EXPLAIN to get a detailed breakdown of how MySQL will execute this query. For example, in your terminal or SQL client:

SQL
EXPLAIN SELECT * FROM orders WHERE order_date > '2023-01-01';
Step 3: Analyze the Output

The output from EXPLAIN will provide several important pieces of information:

a. Type: Indicates how MySQL retrieves rows from the table (e.g., ALL, ref, range, index, etc.).
ALL means that MySQL has to read through each row in the table, which can be very slow.
ref and eq_ref indicate index usage.
b. Possible Keys: Lists all indexes available for the query.
c. Key: The actual key used by the query (if any).
d. Key Length: How many columns in the chosen index are being compared.
e. Rows Examined: The number of rows that MySQL needs to read from the disk to retrieve your results.
f. Rows Sent: The number of rows sent to the client after filtering by the WHERE clause and other conditions.
Step 4: Interpret the Output

Let’s say the output looks like this:

PLAINTEXT
+----+-------------+-------+------------+------+-----------------+-------------+---------+----------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys   | key         | key_len | ref            | rows | filtered | Extra       |
+----+-------------+-------+------------+------+-----------------+-------------+---------+----------------+------+----------+-------------+
|  1 | SIMPLE      | orders| NULL       | ALL  | order_date_idx  | NULL        | NULL    | NULL           | 1000 |   100.00 | Using where |
+----+-------------+-------+------------+------+-----------------+-------------+---------+----------------+------+----------+-------------+

Here, we can see that:

The type column shows ALL, indicating a full table scan.
The possible_keys and key columns show order_date_idx, but it’s not being used.
Step 5: Improve the Query

To improve this query, consider adding an index on the order_date column:

SQL
CREATE INDEX idx_order_date ON orders(order_date);

Then re-run EXPLAIN to see if the execution plan has improved:

PLAINTEXT
+----+-------------+-------+------------+------+-----------------+-------------+---------+----------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys   | key         | key_len | ref            | rows | filtered | Extra       |
+----+-------------+-------+------------+------+-----------------+-------------+---------+----------------+------+----------+-------------+
|  1 | SIMPLE      | orders| NULL       | range| idx_order_date  | idx_order_date| 4        | NULL           |   25 |    83.33 | Using where |
+----+-------------+-------+------------+------+-----------------+-------------+---------+----------------+------+----------+-------------+

Now, we see that type is range, indicating an index scan which is more efficient.

Step 6: Validate the Changes

Run the query again to verify the performance improvement:

SQL
EXPLAIN SELECT * FROM orders WHERE order_date > '2023-01-01';

You should see a significant reduction in rows examined and possibly an index type.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using EXPLAIN effectively helps you understand the execution plan of your queries. By analyzing the output, you can identify and optimize bottlenecks like full table scans, inappropriate index usage, or missing indexes. Key takeaways include:

Always check the type column to ensure that an appropriate index is being used.
Use EXPLAIN for both initial query analysis and ongoing performance tuning.

#### Common mistakes
to avoid:

Ignoring Indexes: Not understanding how indexes can significantly improve query performance.
Over-indexing: Creating too many indexes, which can degrade write performance without providing substantial benefits.
Misinterpreted type Values: Misunderstanding the implications of different type values (e.g., ALL, range, index).

#### Follow-ups
might include:

Discussing additional optimization techniques like partitioning or query rewriting.
Exploring alternative database solutions if MySQL’s performance still doesn’t meet requirements.

