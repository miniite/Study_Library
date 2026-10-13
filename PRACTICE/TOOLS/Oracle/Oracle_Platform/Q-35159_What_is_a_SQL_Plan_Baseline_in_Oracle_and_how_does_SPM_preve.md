---
type: "practice"
difficulty: "Expert"
---

# What is a SQL Plan Baseline in Oracle and how does SPM prevent plan regressions after a stats or upgrade change?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Oracle SQL and Indexes](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_SQL_and_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A SQL Plan Baseline (SPB) in Oracle Database is a feature used to manage and control execution plans of frequently executed SQL statements. The SQL Performance Management (SPM) process automates the creation and maintenance of SPBs, ensuring that query performance remains consistent even after schema changes or upgrades.

#### Triaging the Question

The question revolves around understanding how SQL Plan Baselines are utilized in Oracle Database to manage query performance. Specifically, it asks about SPM's role in preventing plan regressions when there are statistics updates or database upgrades. Here’s what we need to cover:

Understanding SQL Plan Baselines (SPBs): What they are and their purpose.
Role of SPM: How SPM manages and maintains SPBs to ensure performance stability.
Impact of Schema Changes and Upgrades: Why these changes can affect query plans, and how SPM addresses this issue.
Detailed Solution:
1. Understanding SQL Plan Baselines (SPBs):

SQL Plan Baselines are a set of execution plans that Oracle stores in the database to track historical performance. They allow DBAs to control which execution plan is used for a specific SQL statement based on criteria such as optimizer statistics, index usage, and other factors.

When a query runs with the OPTIMIZER_USE_PLAN_BASELINE hint or when SPM automatically identifies an efficient plan, Oracle creates a new entry in the SPB. This entry stores information about the execution plan, including its performance metrics like CPU time, physical reads, and sorts.

2. Role of SQL Performance Management (SPM):

SQL Performance Management is an automated feature within Oracle Database that manages SPBs to ensure optimal query performance. SPM works by regularly analyzing execution plans and comparing them against existing baseline plans. When it detects a better plan or when manual hints are used, SPM creates new SPB entries.

Key Steps in SPM:

Analysis: SPM continuously monitors the database for SQL statements that are candidates for plan management.
Auto-Tuning: It periodically reoptimizes the identified statements and stores any improved plans as SPBs.
Baseline Selection: During query execution, Oracle uses SPM to select the best plan from the available baselines.

SPM Process:

Statement Analysis: SPM examines frequently executed SQL statements.
Plan Evaluation: It evaluates new or existing plans based on performance metrics.
Plan Baseline Creation: If a better plan is identified, it is added to the SPB.
3. Impact of Schema Changes and Upgrades:

Schema changes and database upgrades can alter the execution environment in ways that invalidate previous execution plans. For example:

Statistics Updates: When statistics are updated due to new data or changes in the distribution of values, the optimizer may choose different execution paths.
Index Modifications: Adding or removing indexes can change how queries are executed.

These changes can lead to suboptimal performance if the query plan no longer matches the current workload. This is where SPM plays a crucial role:

Reoptimization on Upgrade: After an upgrade, SPM reoptimizes SQL statements and stores new plans in SPBs.
Plan Stability: By maintaining multiple baselines, SPM ensures that even after changes, the most efficient plan can be selected.

#### Closing Summary

SQL Plan Baselines are a powerful tool for managing query execution plans in Oracle Database. The SQL Performance Management process automates the maintenance of these baselines to ensure performance stability. It addresses issues like schema changes and upgrades by reoptimizing statements and maintaining multiple baseline plans, thereby preventing plan regressions.

#### Common Mistakes

Not enabling SPM or not regularly managing SPBs can lead to suboptimal query performance.
Overlooking the need for regular statistics updates, which can invalidate existing baselines.

#### Follow-Up
Questions:
How does Oracle determine which execution plan is selected from multiple SPB entries?
What are the trade-offs between using SPM and manually tuning queries?

**Comparison:**

| SQL_ID | Plan_Hash_Value | Plan_Name         | Estimated_Cost | Actual_Time |
|--------|-----------------|------------------|---------------|-------------|
| abc123 | 1234567890      | Baseline Plan     | 123.45        | 234.56      |
| def456 | 2345678901      | Alternate Plan    | 125.00        | 240.00      |

**Code:**

```sql
-- Example of creating a SQL Plan Baseline
BEGIN
  DBMS_SPM.SET_SCRIPT_EXECUTION_BUILDER_FLAGS('SQL_ID = "abc123" AND PLAN_HASH_VALUE = 1234567890');
END;
/
```

**Example input / output:**

```text
Input:
SELECT * FROM employees WHERE department_id = 10;

Output (with SPM and SPBs):
Plan 1: Using index access path, costing 123.45 and taking 234.56 seconds.
Plan 2: Using full table scan, costing 120.00 and taking 220.00 seconds.

Selected Plan:
Plan 2 (full table scan) due to better performance metrics after statistics update.
```

**Data:**

```json
{
  "sql_id": "abc123",
  "plan_hash_value": 1234567890,
  "plan_name": "Baseline Plan for Query abc123",
  "estimated_cost": 123.45,
  "actual_time": 234.56
}
```

**Flow:**

1. Yes
2. No
3. SQL Statement
4. SPM Analysis
5. Is Plan Better?
6. Store New Baseline
7. Use Existing Baseline
8. Monitor Performance
9. Monitor Performance

