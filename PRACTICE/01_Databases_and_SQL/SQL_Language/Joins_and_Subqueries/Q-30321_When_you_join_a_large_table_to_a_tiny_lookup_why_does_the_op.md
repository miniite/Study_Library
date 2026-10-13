---
type: "practice"
difficulty: "Deep"
---

# When you join a large table to a tiny lookup, why does the optimizer usually build the hash on the small side, and when is that wrong?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ EXISTS Semi-joins and Predicate Placement](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/EXISTS_Semi-joins_and_Predicate_Placement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The database optimizer typically builds the hash in the smaller table during joins because it reduces the overall memory footprint, speeds up execution time, and minimizes lock contention. However, there are scenarios where this default behavior might be suboptimal.

#### Triaging the Question
What is being asked: You're probing the reasoning behind the optimizer's decision to build a hash on the smaller table during join operations.
Key concepts involved: Joins, subqueries, database optimization, memory management, and execution plans.

#### Assumptions
The tables are significantly different in size (large vs. small).
We're discussing hash joins.

#### First Principles Solution
Why the Optimizer Prefers Smaller Tables for Hash Joins

Memory Management:

A hash join requires storing all rows of one table in memory to build a hash table, which can be significant if the table is large.
Storing data from the smaller (lookup) table helps keep overall memory usage lower and avoids potential out-of-memory errors.

Execution Time:

Hash joins have linear time complexity for both building the hash table and searching it, making them more efficient when dealing with a small number of rows.
Smaller tables result in faster build times and quicker lookups during join operations.

Lock Contention:

Holding locks on smaller tables can be less resource-intensive compared to larger ones, reducing lock contention and improving concurrency.
If both tables are large, the optimizer might choose other strategies (like nested loops) instead of hash joins due to higher memory and locking requirements.
When Building Hash on the Larger Table Might Be More Appropriate

Memory Availability:

If there is plenty of available memory, it may be more beneficial to build the hash on the larger table to reduce the number of join operations.
This can also simplify query execution if the smaller table has complex conditions or needs extensive processing.

Join Conditions and Filters:

If the larger table contains many null values or a high percentage of non-matching rows, building the hash there can improve performance by reducing the amount of data being processed.
Conversely, if the smaller table is highly selective (many matching rows), building the hash there can be more efficient.

Query Complexity:

In complex queries with multiple joins or subqueries, it might make sense to build hashes on both tables based on the overall plan and cost estimation.
The optimizer might choose a different approach if additional factors like nested loops or merge joins offer better performance for specific use cases.

Database Size and Performance:

In large-scale databases where memory is a critical resource, minimizing the memory footprint of hash builds can be crucial.
However, in smaller environments with ample resources, building hashes on both tables could provide more balanced and efficient execution plans.

#### Closing Summary
and Common Mistakes

Common Misunderstandings:

Overreliance on the default behavior without considering context-specific factors.
Not recognizing that memory availability can significantly impact join optimization strategies.

#### Follow-Up
Questions:

How does this change with different database management systems (e.g., MySQL, PostgreSQL)?
What are the implications for distributed databases or cloud environments?
Can you provide an example of when building a hash on the larger table would be beneficial?
Summary

The optimizer typically builds hashes on smaller tables during joins to optimize memory usage and execution time. However, there are scenarios where this default behavior can lead to suboptimal performance, particularly in large-scale or resource-constrained environments.

**Comparison:**

```
large_table	Column A	Column B	Key
...	...	...	...
small_lookup	Key	Value
...	...	...
```

**Code:**

```sql
-- Example SQL for a join operation
SELECT *
FROM large_table l
JOIN small_lookup s ON l.key = s.key;
```

**Example input / output:**

```text
-- Input Example
JOIN operation between large_table (10M) and small_lookup (1K)

-- Output Example
Execution Plan:
- Hash build on small_lookup table.
- Hash lookup on large_table with the hash table built from small_lookup.

Memory Usage:
- Small Lookup: 4MB (stored in hash)
- Large Table: 8GB (processed row by row)
```

**Data:**

```text
- `large_table`: 10 million rows, with various data types and complex conditions.
- `small_lookup`: 1000 rows, all unique values for a specific key.
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Small Table Smaller?
5. Build Hash on Small Table
6. Consider Other Join Strategies
7. Join Operation ExecutedEfficiently
8. Opt for Nested Loops orMerge Joins

