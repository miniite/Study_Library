---
type: "practice"
difficulty: "Deep"
---

# What does 'Using filesort' in EXPLAIN mean and how would you eliminate it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address "Using filesort" in an EXPLAIN plan, we need to understand what filesorting is, when MySQL uses it, and how we can avoid it. Filesorting refers to a process where MySQL needs to sort data after reading from disk but before returning results to the user. This can be inefficient because it involves additional I/O operations.

TL;DR

"Using filesort" in EXPLAIN indicates that MySQL will perform an extra sorting step on the result set, which is costly and should be avoided if possible. We can eliminate this by ensuring proper indexing and using appropriate query optimizations.

#### Triage
What's Being Probed?

The question aims to test your understanding of MySQL query optimization, specifically around sorting operations and their impact on performance. The interviewer is probing whether you:

Understand the concept of filesorting.
Know when and why MySQL uses it.
Can identify and correct inefficient queries.

#### First-Principles Solution
What is Filesorting?

Filesorting occurs in scenarios where a query's ORDER BY clause cannot use an index. Instead, MySQL reads the rows from disk, sorts them in memory, and then returns the result to the user. This process can be slow due to additional I/O operations.

When Does MySQL Use Filesort?

MySQL uses filesort when:

No Index is Used: If no suitable index exists for the ORDER BY clause.
Column Types: The data types in the ORDER BY clause are not supported by indexes (e.g., functions, expressions).
Complex Queries: Queries with multiple tables and joins can also trigger filesort if proper indexing is lacking.

#### Example

#### Cons
ider a table orders:

SQL
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10, 2)
);

A query like:

SQL
SELECT * FROM orders ORDER BY customer_id, amount;

If there is no index on customer_id or amount, MySQL might use filesort.

Optimizing the Query

To eliminate filesort, we can take several steps:

Create Indexes: Ensure that indexes exist for columns used in ORDER BY.
Covering Indexes: Create an index that covers all columns needed by the query.
Avoid Functions and Expressions: Do not use functions or expressions on indexed columns in the ORDER BY clause.

#### Example
Solution

Adding an appropriate index:

SQL
CREATE INDEX idx_customer_id_amount ON orders (customer_id, amount);

This index allows MySQL to directly access the required data without performing filesort.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, "Using filesort" in EXPLAIN indicates potential performance issues. To eliminate it:

Indexing: Ensure that indexes cover the columns used in ORDER BY.
Covering Indexes: Optimize by creating an index that covers all necessary columns.
Avoid Functions and Expressions: Do not use functions or expressions on indexed columns.

#### Common mistakes
include ignoring query optimization, overusing subqueries, and failing to understand the impact of indexing on performance.

Common Follow-Ups
What other types of sorting are used by MySQL?
How does filesort compare with MERGE JOIN in terms of performance?
Are there any performance implications if a query uses an index but still requires filesort?

**Comparison:**

```
id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
1	SIMPLE	orders	NULL	ALL	NULL	NULL	NULL	NULL	3	100.00	Using filesort
id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
2	SIMPLE	orders	NULL	const	idx_customer_id_amount	idx_customer_id_amount	8	const	1	100.00	Using index
```

**Code:**

```sql
-- Example SQL for creating indexes
CREATE INDEX idx_customer_id_amount ON orders (customer_id, amount);
```

**Example input / output:**

```text
-- Initial query with filesort
EXPLAIN SELECT * FROM orders ORDER BY customer_id, amount;

-- Optimized query without filesort
EXPLAIN SELECT * FROM orders WHERE customer_id = 102 ORDER BY amount;
```

**Data:**

```sql
-- Sample table schema
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10, 2)
);

-- Insert sample data
INSERT INTO orders VALUES
(1, 101, '2023-01-01', 50.00),
(2, 102, '2023-01-02', 75.00),
(3, 103, '2023-01-03', 100.00);
```

**Flow:**

1. Query with filesort
2. Uses additional I/O for sorting
3. Slow performance
4. Index exists but not coveringall needed columns
5. Consider creating a coveringindex
6. Using functions or expressionsin ORDER BY
7. Avoid using functions orexpressions on indexedcolumns

