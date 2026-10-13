---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would use EXPLAIN (GENERIC_PLAN) or plan-only tooling to tune a query without executing it.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Reading Execution Plans](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Reading_Execution_Plans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively use EXPLAIN (GENERIC_PLAN) or similar plan-only tools for tuning queries in a database, you first need to understand how these tools work and what they reveal about your SQL execution plans. This approach allows you to optimize the query structure and parameters without actually running it, which is particularly useful when dealing with complex or resource-intensive queries.

#### Triaging the Question

The question is asking how to use EXPLAIN (GENERIC_PLAN) for query tuning without executing the query. The interviewer likely wants to see your understanding of:

How EXPLAIN works and its output.
Techniques to interpret the execution plan.
Strategies to improve performance based on the plan.

#### Assumptions
You are working with a SQL database that supports EXPLAIN or an equivalent tool.
The query in question is complex enough to require optimization, but not so resource-intensive that you can't afford to analyze it fully.

#### First-Principles Solution
Step 1: Understanding the EXPLAIN Output

The EXPLAIN (GENERIC_PLAN) output provides a detailed breakdown of how the database engine plans to execute your query. It typically includes:

The order in which tables are scanned.
Indexes used, if any.
Join types and join orders.
Estimated costs for each operation.
Step 2: Analyzing the Execution Plan

You should start by examining the output for clues on how the database engine intends to execute the query. For example:

Table Scan vs. Index: Determine whether full table scans or indexes are being used. Full table scans can be very expensive, especially if the tables are large.
Join Order and Type: The order of joins and the type (e.g., NATURAL, INNER, LEFT JOIN) can significantly impact performance. Some join orders may lead to more efficient execution plans.
Cost Estimates: Look at the cost estimates provided by the database. These are relative costs based on the query optimizer's understanding, helping you prioritize which parts of the plan need optimization.
Step 3: Identifying Bottlenecks

Based on the EXPLAIN output, identify any potential bottlenecks:

Heavy table scans.
Poorly chosen join orders or types.
Missing indexes that could improve performance.
Step 4: Proposing Improvements

Once you've identified the issues, suggest improvements such as:

Creating or Reordering Indexes: Adding new indexes or altering existing ones to better support your query’s access patterns.
Revising Join Orders and Types: Changing the order in which tables are joined can sometimes lead to more efficient execution plans.
Query Rewriting: Sometimes, rewriting parts of the query can result in a different and potentially more optimal execution plan.
Step 5: Verifying with EXPLAIN Again

After making changes, run EXPLAIN (GENERIC_PLAN) again to verify that your proposed optimizations have improved the execution plan. Keep refining until you achieve an acceptable performance level.

#### Closing Summary
and Common Mistakes/Follow-Ups

Using EXPLAIN (GENERIC_PLAN) effectively requires a deep understanding of how database engines work and the ability to interpret their output accurately. Some common mistakes include:

Overlooking indexes that could significantly reduce scan costs.
Ignoring join order, which can have substantial performance impacts.
Focusing too much on individual operations without considering the overall plan.

For follow-ups, you might be asked about specific tools or methods for index management, how to handle subqueries and correlated queries, or even more advanced topics like materialized views and query rewriting strategies.

**Comparison:**

```
customer_id	first_name	last_name	country
1	John	Doe	USA
order_id	customer_id	product_id
10	1	500
```

**Code:**

```sql
-- Example SQL Query
SELECT *
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.country = 'USA';
```

**Example input / output:**

```
Example EXPLAIN output:
PLAINTEXT
Copy
"EXPLAIN PLAN FOR SELECT * FROM customers c JOIN orders o ON c.customer_id = o.customer_id WHERE c.country = 'USA'"

"Query Plan:
 - Table Scan on customers (cost=100)
 - Table Scan on orders (cost=200)
 - Join: c.customer_id = o.customer_id
 - Filter: c.country = 'USA'"
```

**Data:**

```
Sample table schema:
SQL
Copy
CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50),
  country VARCHAR(50)
);

CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT,
  product_id INT,
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

**Flow:**

1. Query
2. EXPLAIN (GENERIC_PLAN)
3. Analyze Plan
4. Identify Bottlenecks
5. Propose Improvements
6. Verify with EXPLAIN
7. Optimized Query

