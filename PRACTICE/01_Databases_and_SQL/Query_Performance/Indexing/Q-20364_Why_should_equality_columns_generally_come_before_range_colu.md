---
type: "practice"
difficulty: "Deep"
---

# Why should equality columns generally come before range columns in a composite index used by an ETL filter query?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Composite Indexes and Column Order](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Composite_Indexes_and_Column_Order.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a composite index for a database, it's crucial to understand how SQL engines use indexes for query optimization. In a composite index, equality columns should generally be placed before range columns because this order allows the database to more efficiently prune the search space during execution. This answer delves into the reasons behind this best practice and provides a step-by-step explanation from first principles.

Firstly, let's triage the question: The core of the query is about why equality conditions should come before range conditions in composite indexes used by ETL filter queries. The interviewer is probing your understanding of how SQL engines leverage indexes for performance optimization. We'll assume that you are dealing with a relational database and an index designed to optimize read operations, such as those performed during ETL processes.

Now, let's break down the reasoning from first principles:

Equality vs Range Conditions: Equality conditions (=, IN) return a single or very few rows, while range conditions (e.g., <, >, BETWEEN) can return many rows.
Index Scanning: When an index is used, the database engine scans the index to find matching rows. The order in which columns are listed affects how many rows are examined.

When equality columns come first:

The index can quickly narrow down the search space by using the equality condition(s). For example, if a composite index on (A, B) is created and A has an equality condition (e.g., A = 10), the database engine will look for all rows where A = 10. This significantly reduces the number of rows that need to be examined.
After finding matching rows based on A, the index can then be used to quickly find the corresponding values in column B. Since the search space is much smaller, this step becomes more efficient.

In contrast:

If range conditions come first (e.g., (B < 10, A), the database engine will have to scan a potentially large set of rows based on B before applying any equality condition. This increases the number of rows that need to be examined and processed.

To illustrate with an example: Consider a table with columns customer_id (equality), transaction_date (range), and amount. An index on (customer_id, transaction_date) would allow the database to quickly identify all transactions for a specific customer (customer_id = 10) and then efficiently filter by date. This is more efficient than an index on (transaction_date, customer_id), where a large number of rows might need to be examined before filtering by customer_id.

Here’s a step-by-step breakdown:

Index: (customer_id, transaction_date)
Query: SELECT * FROM transactions WHERE customer_id = 10 AND transaction_date BETWEEN '2023-01-01' AND '2023-01-31'
Execution Plan:
Step 1: The index is used to find all rows where customer_id = 10. This step involves scanning a small set of rows.
Step 2: The date range condition (transaction_date BETWEEN '2023-01-01' AND '2023-01-31') is applied to the subset of rows returned from step 1. Since fewer rows are being considered, this step is faster.

In conclusion, placing equality columns before range columns in a composite index optimizes query performance by reducing the number of rows that need to be scanned and processed. This best practice should always be followed when designing indexes for ETL processes or any read-heavy operations where filter queries are common.

**Code:**

```sql
CREATE INDEX idx_customer_date ON transactions (customer_id, transaction_date);
```

