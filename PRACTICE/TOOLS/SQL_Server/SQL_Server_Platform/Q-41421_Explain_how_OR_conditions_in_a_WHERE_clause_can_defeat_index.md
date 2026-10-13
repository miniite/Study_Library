---
type: "practice"
difficulty: "Intermediate"
---

# Explain how OR conditions in a WHERE clause can defeat index usage, and how UNION ALL or a rewrite can restore seeks.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ OR Conditions and Index Usage](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/OR_Conditions_and_Index_Usage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL Server, using OR conditions in a WHERE clause can sometimes prevent the database engine from utilizing indexes effectively. This is because SQL Server may choose to perform a table scan instead of a seek operation when faced with complex conditions involving OR. However, there are ways to rewrite queries or use UNION ALL to restore efficient index usage.

#### Triaging the Question

The core of this question revolves around understanding how query optimization works in SQL Server and recognizing specific scenarios where indexes cannot be used effectively. The interviewer is likely probing your knowledge of query execution plans, indexes, and alternative methods for leveraging them.

#### Assumptions
You have a basic understanding of SQL Server indexing and query execution.
You are familiar with the difference between OR conditions and how they impact plan choice.
You know about UNION ALL and its role in achieving better performance.

#### First-Principles Solution

Let's start by looking at an example where using OR can lead to suboptimal performance:

SQL
SELECT *
FROM Orders
WHERE OrderDate > '2021-01-01' OR CustomerID = 123;

In this query, the database engine may decide that a table scan is faster than performing index seeks for each condition. This is because SQL Server cannot easily merge the results of two separate index lookups into one result set.

To understand why, consider how an OR condition in a WHERE clause works:

The database engine evaluates the first part (OrderDate > '2021-01-01') and then the second part (CustomerID = 123).
Without knowing which part will return fewer rows, it might choose to scan the entire table.
Using UNION ALL

One effective way to rewrite such queries is by using UNION ALL. This allows you to separate the conditions into multiple, simpler queries that can utilize indexes more effectively:

SQL
SELECT *
FROM Orders
WHERE OrderDate > '2021-01-01'
UNION ALL
SELECT *
FROM Orders
WHERE CustomerID = 123;

By splitting the query this way:

The first part (OrderDate > '2021-01-01') can leverage an index on OrderDate.
The second part (CustomerID = 123) can use an index on CustomerID.

This approach ensures that both conditions are checked separately, allowing the database engine to perform seeks rather than scans.

Alternative: Rewriting the Query

Another method involves rewriting the query using a CASE statement or other logical constructs. For instance:

SQL
SELECT *
FROM Orders
WHERE (OrderDate > '2021-01-01' AND CustomerID = 123)
   OR OrderDate <= '2021-01-01';

This version checks the CustomerID first and then applies the OrderDate condition only if needed. However, this might not always be the most efficient approach due to potential order of operations issues.

#### Closing Summary

In summary, using OR conditions in a WHERE clause can prevent SQL Server from utilizing indexes effectively. To restore index usage, you can use UNION ALL to split the query into simpler parts or rewrite the query with logical constructs. Understanding these techniques is crucial for optimizing performance and ensuring efficient execution plans.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the potential impact of OR conditions on index usage.

#### Follow-Up
Exploring more complex scenarios involving multiple columns and indexes, or discussing the performance implications of different rewrites.

**Comparison:**

```
No
Yes
Yes
Original Query
Does Index Utilization Work?
Full Table Scan
Using UNION ALL
Can Both Conditions BeSatisfied Efficiently?
Efficient Execution Plan
Alternative Rewrite with CASEStatement
Does It Simplify Index Usage?
Easier to Implement EfficientIndexes
```

**Code:**

```sql
-- Original query
SELECT *
FROM Orders
WHERE OrderDate > '2021-01-01' OR CustomerID = 123;

-- Using UNION ALL
SELECT *
FROM Orders
WHERE OrderDate > '2021-01-01'
UNION ALL
SELECT *
FROM Orders
WHERE CustomerID = 123;

-- Alternative rewrite with CASE statement
SELECT *
FROM Orders
WHERE (OrderDate > '2021-01-01' AND CustomerID = 123)
   OR OrderDate <= '2021-01-01';
```

**Example input / output:**

```text
| OrderID | OrderDate       | CustomerID |
|---------|----------------|------------|
|        1| 2021-02-01     |          1 |
|        2| 2023-01-01     |      123   |

Original query output:
- OrderID: 1, OrderDate: 2021-02-01 (full table scan)
- OrderID: 2, OrderDate: 2023-01-01 (full table scan)

Using UNION ALL:
- OrderID: 1, OrderDate: 2021-02-01
- OrderID: 2, OrderDate: 2023-01-01

Alternative rewrite with CASE statement output:
- OrderID: 1, OrderDate: 2021-02-01 (uses `OrderDate` index)
- OrderID: 2, OrderDate: 2023-01-01 (full table scan)
```

**Data:**

```json
{
  "Orders": [
    { "OrderID": 1, "OrderDate": "2021-02-01", "CustomerID": 1 },
    { "OrderID": 2, "OrderDate": "2023-01-01", "CustomerID": 123 }
  ]
}
```

**Flow:**

1. +--------------------------------------------------------------------+
2. | Original Query                                                      |
3. | SELECT *                                                            |
4. | FROM Orders                                                         |
5. | WHERE OrderDate > '2021-01-01' OR CustomerID = 123                   |
6. +---------------------+-------------------------------------------+
7. |                                            |
8. +------v------+                             |
9. | Full Table Scan                    +-------v-------+
10. | (No Index Utilization)           | UNION ALL   |
11. +------^------+                     +--------^-----+
12. |              |                     |         |
13. +-------------+  +----------+-----------------+  +------------+
14. | Using UNION ALL                                   | Alternative Rewrite with CASE Statement
15. | SELECT *                                          | SELECT *
16. | FROM Orders                                       | FROM Orders
17. | WHERE OrderDate > '2021-01-01'                    | WHERE (OrderDate > '2021-01-01'
18. | UNION ALL                                         |   AND CustomerID = 123)
19. | SELECT *                                          | OR OrderDate <= '2021-01-01')
20. | FROM Orders                                       |
21. | WHERE CustomerID = 123                            |
22. +-------------+  +----------+-----------------+
23. |              |
24. +------v------+         |
25. | Efficient Indexes     |
26. | Utilization           |
27. +---------------------+

