---
type: "practice"
difficulty: "Deep"
---

# What is the difference between the ORDER BY SORT BY DISTRIBUTE BY and CLUSTER BY clauses in Hive?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Hive Partitioning and Bucketing](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/Hive_Partitioning_and_Bucketing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Hive, the ORDER BY, SORT BY, DISTRIBUTE BY, and CLUSTER BY clauses are used to control how data is sorted and distributed across partitions or nodes. However, they serve different purposes and can have significant impacts on query performance. A fresh understanding of these clauses will help you write more efficient Hive queries.

The core idea here is to understand the role each clause plays in controlling data distribution and sorting:

ORDER BY is used at the end of a SELECT statement to sort the final result set.
SORT BY is an older clause used within UDAFs (User Defined Aggregation Functions) and is deprecated but still supported for backward compatibility.
DISTRIBUTE BY controls how data is distributed across different nodes in a cluster, ensuring that related rows are on the same node.
CLUSTER BY is similar to DISTRIBUTE BY, but it performs an additional sort before shuffling.

Let's break down each clause with a first-principles approach:

ORDER BY: This clause sorts the final result set after all operations (like joins, filters, etc.) have been applied.

SQL
SELECT * FROM table ORDER BY column;

SORT BY: This was used in older versions of Hive within UDAFs to sort intermediate results before aggregations. However, it has largely been replaced by the newer SORT BY functionality available directly on SELECT statements starting from Hive 0.13.

SQL
SELECT SUM(column) SORT BY column;

DISTRIBUTE BY: This clause is used to distribute data across nodes in a way that ensures related rows are processed together, which can optimize join operations and improve query performance.

SQL
SELECT * FROM table1 JOIN table2 ON key DISTRIBUTE BY key;

CLUSTER BY: Similar to DISTRIBUTE BY, but it performs an additional sort operation before the shuffle process. This is useful when you want to ensure that rows are not only distributed across nodes but also sorted within each node.

SQL
SELECT * FROM table1 JOIN table2 ON key CLUSTER BY key;

To summarize, ORDER BY sorts the final result set, while DISTRIBUTE BY and CLUSTER BY control how data is initially distributed across nodes. The choice between them depends on whether you want to perform an additional sort (CLUSTER BY) or just distribute without sorting.

Now, let's consider a few common mistakes:

Using ORDER BY instead of SORT BY in UDAFs.
Not understanding the difference between DISTRIBUTE BY and CLUSTER BY, leading to suboptimal query performance.
Overusing these clauses without considering the actual data distribution strategy.

