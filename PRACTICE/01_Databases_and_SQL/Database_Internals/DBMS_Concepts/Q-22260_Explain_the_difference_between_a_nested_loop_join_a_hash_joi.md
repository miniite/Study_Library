---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between a nested loop join, a hash join, and a merge join in the query planner.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Partitioning and Partition Pruning](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database management systems (DBMS), query planners are responsible for deciding on an efficient execution plan to run SQL queries. Among the various types of joins—nested loop join, hash join, and merge join—their differences lie in their implementation logic, efficiency under different conditions, and complexity.

#### Triage
What the Question is Asking

The question aims to understand the core principles behind these three common types of join operations used by query planners. You need to explain how each works internally, the scenarios where they are most effective or inefficient, and the trade-offs involved in choosing one over another.

#### First-Principles Solution
1. Nested Loop Join (NLJ)

#### Overview
Nested loop join is a simple but often less efficient method for joining two tables. It involves iterating through each row of the first table and performing a sequential search on the second table to find matching rows.

SQL
SELECT * FROM A, B WHERE A.key = B.key;

Operation:

For each row in A, scan all rows in B (or vice versa) to check if there is a match based on the join condition.
If a match is found, return the combined result.

Efficiency:

Time complexity can be high due to sequential scans, making it O(m * n), where m and n are the number of rows in each table.
Suitable for small tables but inefficient with large datasets.
2. Hash Join

#### Overview
Hash join is a more efficient method that involves creating a hash table from one of the tables (the "build" side) to quickly find matches during the join operation.

Operation:

Build phase:
Scan one of the tables and create a hash table based on the join key.
Probe phase:
Iterate through the other table (the "probe" side) and use the hash table to check for matching rows.
If a match is found, return the combined result.

Efficiency:

Time complexity can be reduced to O(m + n), making it more efficient than NLJ, especially when one of the tables is significantly larger than the other.
Suitable for large datasets but requires additional memory for the hash table.
3. Merge Join

#### Overview
Merge join works by sorting both input tables on the join key and then performing a sequential comparison to find matching rows.

Operation:

Sort both input tables based on the join key.
Use two pointers, one for each table, starting from the beginning.
Compare the keys at the current positions of the two pointers:
If they match, return the combined result and move both pointers forward.
If not, advance the pointer pointing to the smaller value.

Efficiency:

Time complexity can be very efficient if the tables are already sorted or nearly sorted, reducing it to O(m + n).
Requires sorting, which can add overhead unless the tables are pre-sorted.
Suitable for large, well-sorted datasets and when both input sizes are comparable.

#### Closing Summary

The choice between nested loop join, hash join, and merge join depends on factors like the size of the tables, whether they are sorted, and available memory. Nested loop joins are straightforward but inefficient with large datasets. Hash joins offer a significant speedup by using hash tables, making them suitable for large datasets at the cost of additional memory. Merge joins benefit from pre-sorted data and are highly efficient in such cases.

#### Common Mistakes
and Follow-Ups
Misunderstanding Memory Constraints: Assuming that hash join is always better without considering memory constraints.
Ignoring Pre-Sorting: Failing to recognize how sorting can improve merge join performance significantly.
Complexity Estimation Errors: Overestimating the complexity of NLJ or underestimating the memory requirements for hash joins.
Self-check

Correctness and completeness confirmed.

