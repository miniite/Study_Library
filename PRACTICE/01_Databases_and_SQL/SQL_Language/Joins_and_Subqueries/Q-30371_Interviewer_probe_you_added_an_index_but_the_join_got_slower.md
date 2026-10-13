---
type: "practice"
difficulty: "Deep"
---

# Interviewer probe: you added an index but the join got slower — how can a new index mislead the optimizer into a worse join method?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ EXISTS Semi-joins and Predicate Placement](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/EXISTS_Semi-joins_and_Predicate_Placement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding an index to improve performance, it's crucial to understand that the database query optimizer is responsible for choosing the best execution plan. In some cases, adding an index might not always lead to a more efficient join method, and sometimes can even degrade performance if the optimizer makes a suboptimal choice. Let's explore why this happens and how we can diagnose and mitigate such issues.

#### Triage
Understanding the Problem

The core issue here is that adding an index can mislead the query optimizer into choosing a less efficient execution plan for the join operation. This problem often arises when the optimizer incorrectly estimates the cost of using certain plans due to changes in the statistics or schema of the database.

#### Assumptions
and Context
Index Impact: We assume that the added index did not improve performance and might have led to a slower query.
Query Optimization: The query optimizer's decisions are based on cost estimation, which can be influenced by various factors such as index usage, statistics, and cardinality estimates.

#### First Principles Solution
Diagnosing and Resolving Index Misleading Issues

Understanding the Query Plan

Use EXPLAIN or equivalent commands to analyze the query execution plan before and after adding the index.
SQL
EXPLAIN SELECT * FROM tableA JOIN tableB ON tableA.id = tableB.id;

Reviewing Cost Estimates

The optimizer's cost estimates are crucial in determining which execution plan is chosen. If the new index changes these estimates, it can lead to a less efficient plan.
SQL
EXPLAIN ANALYZE SELECT * FROM tableA JOIN tableB ON tableA.id = tableB.id;

Checking Index Usage

Ensure that the optimizer is actually using the newly added index as intended.
Use EXPLAIN with the ANALYZE option to get detailed execution statistics.

Cardinality and Statistics

Verify that the database statistics are up-to-date, as old or inaccurate statistics can lead to poor plan choices.
SQL
VACUUM ANALYZE tableA;

Alternative Indexes and Join Algorithms

#### Cons
ider other join algorithms that might be more suitable given the new index. For example, using hash joins instead of nested loops.
Practical Example: Troubleshooting a Misleading Index

Let's consider a specific scenario where adding an index on tableB.id led to slower performance:

SQL
-- Original Query Plan
EXPLAIN SELECT * FROM tableA JOIN tableB ON tableA.id = tableB.id;

-- New Index Added
CREATE INDEX idx_tableB_id ON tableB(id);

-- Re-run the explain plan
EXPLAIN SELECT * FROM tableA JOIN tableB ON tableA.id = tableB.id;

Upon reviewing the new execution plan, you might notice that the optimizer chose a nested loop join instead of an index-joined or hash joined method. This could indicate that the statistics were not properly updated to reflect the new index.

Mitigation Strategies

Update Statistics

Ensure that the database is aware of the changes in data distribution.
SQL
VACUUM ANALYZE tableB;

Review Query Patterns

Analyze the query patterns and ensure they align with the index usage.

Use Index Hinting

Explicitly hint the optimizer to use or avoid certain indexes if necessary.

Re-evaluate Join Algorithms

Depending on the specific database system, you might have different join algorithms that can be chosen explicitly.

#### Closing Summary
and Common Mistakes

In summary, adding an index can sometimes lead to a slower query execution due to incorrect cost estimation by the optimizer. To resolve such issues, we need to thoroughly analyze the query plan using EXPLAIN, ensure statistics are up-to-date, and consider alternative join algorithms or explicit hints.

#### Common mistakes
include:

Not reviewing the full execution plan with detailed costs.
Assuming that more indexes always lead to better performance without verifying the actual impact.
Overlooking the importance of statistics updates in influencing the optimizer's decisions.

**Comparison:**

```
id == id
TableA
Join
TableB
```

**Code:**

```sql
-- Example SQL queries for explaining and analyzing query plans
EXPLAIN SELECT * FROM tableA JOIN tableB ON tableA.id = tableB.id;
CREATE INDEX idx_tableB_id ON tableB(id);
VACUUM ANALYZE tableB;
```

**Example input / output:**

| tableA.id | tableB.id | tableA.name | tableB.age |
|-----------|-----------|-------------|------------|
|         1 |        1  |    Alice    |       20   |
|         2 |        2  |    Bob      |       25   |

**Data:**

```json
{
  "tableA": {
    "id": ["1", "2", "3"],
    "name": ["Alice", "Bob", "Charlie"]
  },
  "tableB": {
    "id": ["1", "2", "4"],
    "age": [20, 25, 30]
  }
}
```

**Flow:**

1. No
2. Yes
3. Query
4. Is Index Used?
5. Check Statistics
6. Review Join Algorithm
7. HINT INDEX
8. META DATA CHECKS

