---
type: "practice"
difficulty: "Intermediate"
---

# Explain why you must add a tie-breaker column to ORDER BY when using ROW_NUMBER for stable dedup across reruns.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deterministic Deduplication](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Deterministic_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why we need to add a tie-breaker column to ORDER BY when using ROW_NUMBER for stable deduplication, let's break down the problem and solution step by step.

TL;DR:

When using ROW_NUMBER() in SQL, adding a tie-breaker column is necessary because ROW_NUMBER() only handles ties within the same group. Without additional columns to break these ties, results can vary between runs due to arbitrary sorting, leading to unstable deduplication.

#### Triage
and Assumptions:
We're working with a dataset where rows might have duplicate values based on certain columns.
The goal is to uniquely identify each row in a stable manner across multiple runs of the query.
ROW_NUMBER() is being used to assign a unique sequential integer to rows within a partition of a result set.

#### First Principles Solution

Understanding ROW_NUMBER():

ROW_NUMBER() assigns a unique number to rows within a partition of a table, starting at 1 for the first row in each partition.
It's deterministic within a partition but not across multiple runs unless tie-breaker columns are used.

Identifying Ties:

#### Cons
ider a simple dataset where we want to deduplicate based on email and order by timestamp:
PLAINTEXT
email | timestamp
-------|----------
john  | 10:00
john  | 09:55
Without an additional column, the query might return either row as row number 1 if they are processed in a different order.

Adding a Tie-Breaker Column:

A tie-breaker column helps break ties by providing a secondary sorting criterion.
For example, we can add id to ensure stable results:
SQL
SELECT email, timestamp, id,
       ROW_NUMBER() OVER (PARTITION BY email ORDER BY timestamp, id) as rn
FROM your_table;

#### Closing Summary
and Common Mistakes/Follow-Ups:
Common Mistake: Failing to add tie-breaker columns can lead to non-deterministic results, especially when there are ties.

#### Follow-Up
Questions:
How does this solution handle very large datasets?
Can we use other window functions instead of ROW_NUMBER() for deduplication?
Code Example
SQL
-- Sample SQL query with tie-breaker column
SELECT email, timestamp, id,
       ROW_NUMBER() OVER (PARTITION BY email ORDER BY timestamp, id) as rn
FROM your_table;
Self-Check:

Correctness and completeness confirmed.

