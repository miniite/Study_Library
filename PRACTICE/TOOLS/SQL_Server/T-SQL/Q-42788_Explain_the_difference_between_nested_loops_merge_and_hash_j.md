---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between nested loops merge and hash joins, and what row estimates cause the optimizer to pick each.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ CROSS APPLY and OUTER APPLY](<../../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the differences between nested loop join (NLJ), merge join, and hash join in T-SQL, we need to first understand their underlying mechanisms and how SQL Server's query optimizer decides which one to use based on estimated row counts.

TL;DR:
Nested Loop Join (NLJ): Iterates over the smaller table for each row of the larger table.
Merge Join: Requires both tables to be sorted. It compares rows from both tables in parallel, leading to more efficient execution with large datasets.
Hash Join: Uses a hash table to match rows from the two input streams, suitable for large and unsorted datasets.

#### Triage

The question probes your understanding of the join algorithms and their performance characteristics based on row estimates. The query optimizer uses these estimates to determine which join strategy is most efficient.

#### First Principles Solution

Nested Loop Join (NLJ):

Mechanism: For each row in the outer table, it iterates over all rows in the inner table.
Performance: Suitable for small datasets due to its simplicity but can become inefficient with larger tables as it leads to O(m * n) complexity.

Merge Join:

Pre-requisite: Both input tables must be sorted by the join key.
Mechanism: It compares rows from both tables in parallel, moving one row at a time from each table and merging them based on the join condition.
Performance: Highly efficient for large datasets as it leverages sorting to reduce comparison steps.

Hash Join:

Pre-requisite: Only requires an index or an existing hash structure (like in a temporary table).
Mechanism: It creates a hash table from the smaller input stream and then uses this table to look up rows in the larger input stream.
Performance: Suitable for large datasets as it avoids sorting, leading to O(m + n) complexity.
Row Estimates and Optimizer Choices:

The SQL Server query optimizer evaluates row estimates (estimated number of rows returned by each operator) before deciding on the join strategy. Here’s a step-by-step breakdown:

Small Outer Table with Large Inner Table:

If the outer table is significantly smaller than the inner table, NLJ might be chosen because it iterates over fewer rows.

Sorted Tables and Large Datasets:

For large datasets where sorting is feasible, merge join will likely be selected due to its efficiency in handling comparisons.

Large Unsorted Tables with Indexes/Temporary Hash Structures:

If the inner table can be efficiently hashed or if an index exists, hash join might be chosen for its O(m + n) complexity.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Understanding these join strategies is crucial for optimizing T-SQL queries. A common mistake is assuming that NLJ will always perform poorly with large datasets, ignoring the potential efficiency gains from sorted tables in merge joins or indexed/temporally hashed tables in hash joins. Follow-ups might involve discussing scenarios where manual hints could override the optimizer’s decisions and practical implications of each join type in real-world database management.

**Comparison:**

| Join Type | Mechanism | Pre-requisites | Performance |
| --- | --- | --- | --- |
| NLJ | Iterates over the outer table for each row in the inner table. | None | O(m * n) |
| Merge Join | Compares rows from both tables in parallel using a sorted condition. | Both tables must be sorted on join key. | O(m + n) when sorted, otherwise can be inefficient |
| Hash Join | Uses hash table to match rows between two input streams. | Smaller input can be hashed or indexed. | O(m + n), efficient for large datasets |

**Code:**

```sql
-- Example T-SQL for demonstrating join types
SELECT *
FROM Customer c
JOIN Order o ON c.CustomerID = o.CustomerID; -- NLJ

WITH SortedCustomer AS (
    SELECT *
    FROM Customer
    ORDER BY CustomerID
)
, SortedOrder AS (
    SELECT *
    FROM [Order]
    ORDER BY CustomerID
)
SELECT *
FROM SortedCustomer sc
JOIN SortedOrder so ON sc.CustomerID = so.CustomerID; -- Merge Join

WITH HashedCustomer AS (
    SELECT *
    INTO #TempCustomer
    FROM Customer
)
, HashedOrder AS (
    SELECT *
    INTO #TempOrder
    FROM [Order]
)
SELECT *
FROM #TempCustomer tc
JOIN #TempOrder to ON tc.CustomerID = to.CustomerID; -- Hash Join
```

**Example input / output:**

```text
-- Nested Loop Join Example Input-Output
Input:
SELECT *
FROM Customer c
JOIN Order o ON c.CustomerID = o.CustomerID;

Expected Output:
{
  "Result": [
    { "CustomerID": 1, "Name": "John Doe", "OrderID": 101, "Product": "Laptop" },
    { "CustomerID": 2, "Name": "Jane Smith", "OrderID": 102, "Product": "Mouse" }
  ]
}

-- Merge Join Example Input-Output
Input:
WITH SortedCustomer AS (
    SELECT *
    FROM Customer
    ORDER BY CustomerID
), SortedOrder AS (
    SELECT *
    FROM [Order]
    ORDER BY CustomerID
)
SELECT *
FROM SortedCustomer sc
JOIN SortedOrder so ON sc.CustomerID = so.CustomerID;

Expected Output:
{
  "Result": [
    { "CustomerID": 1, "Name": "John Doe", "OrderID": 101, "Product": "Laptop" },
    { "CustomerID": 2, "Name": "Jane Smith", "OrderID": 102, "Product": "Mouse" }
  ]
}

-- Hash Join Example Input-Output
Input:
WITH HashedCustomer AS (
    SELECT *
    INTO #TempCustomer
    FROM Customer
), HashedOrder AS (
    SELECT *
    INTO #TempOrder
    FROM [Order]
)
SELECT *
FROM #TempCustomer tc
JOIN #TempOrder to ON tc.CustomerID = to.CustomerID;

Expected Output:
{
  "Result": [
    { "CustomerID": 1, "Name": "John Doe", "OrderID": 101, "Product": "Laptop" },
    { "CustomerID": 2, "Name": "Jane Smith", "OrderID": 102, "Product": "Mouse" }
  ]
}
```

**Data:**

```json
{
  "Customer": [
    { "CustomerID": 1, "Name": "John Doe" },
    { "CustomerID": 2, "Name": "Jane Smith" }
  ],
  "Order": [
    { "OrderID": 101, "CustomerID": 1, "Product": "Laptop" },
    { "OrderID": 102, "CustomerID": 2, "Product": "Mouse" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Query Plan
8. Is Outer Table Small?
9. Nested Loop Join
10. Are Both Tables Sorted?
11. Merge Join
12. Is Index or Hash StructureAvailable for Inner Table?
13. Hash Join
14. Nested Loop Join

