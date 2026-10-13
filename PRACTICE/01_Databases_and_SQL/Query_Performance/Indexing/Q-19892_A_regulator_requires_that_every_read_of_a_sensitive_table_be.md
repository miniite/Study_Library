---
type: "practice"
difficulty: "Expert"
---

# A regulator requires that every read of a sensitive table be provably bounded in cost to prevent a rogue full-scan exfiltration. How would you architect index and query constraints (mandatory predicates, cost gates) to enforce that at the platform level?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the regulator's requirement of bounding the cost of every read on a sensitive table, we need to ensure that queries accessing this table either use appropriate indexes or are constrained in their execution plan such that they do not degenerate into full scans. We will leverage database indexing and query constraints to enforce this at the platform level.

Firstly, let's consider the key problem: preventing full-scan operations while ensuring efficient reads. A full scan can be extremely costly from a performance perspective, especially on large datasets. To mitigate this risk, we need to ensure that queries accessing sensitive data are directed towards using indexes or at least constrained in their execution plans.

#### Triaging the Question

The question is essentially asking how to architect database indexing and query constraints to enforce bounded cost reads for sensitive tables. The interviewer is likely probing your understanding of:

Indexing strategies (covering indexes, partial indexes)
Query optimization techniques
Cost-based query planning
Platform-level enforcement mechanisms

#### First-Principles Solution

Indexing Strategy: We need to ensure that the most frequently accessed data subsets are covered by efficient indexes. This can be achieved through a combination of:

Primary Key Indexes: To enforce unique constraints and provide fast access to records.
Covering Indexes: Including all necessary columns in the index to avoid disk I/O during query execution.
Partial Indexes: Limiting the data covered by an index to reduce its size and improve performance.

Query Constraints: We must ensure that queries accessing sensitive tables are forced to use specific indexes or are constrained to prevent full scans. This can be achieved through:

Mandatory Predicates: Using WHERE clauses that guarantee efficient execution.
Cost Gates: Implementing query cost limits at the database level to enforce bounded costs.

Platform-Level Enforcement: We need a mechanism to monitor and enforce these constraints at the platform level, ensuring they are consistently applied across all queries.

#### Implementation
Steps

Design Indexes:

SQL
-- Example: A primary key index on a sensitive table `sensitive_data`
CREATE INDEX idx_sensitive_data_id ON sensitive_data (id);

-- Example: A covering index for frequent query patterns
CREATE INDEX idx_sensitive_data_filter ON sensitive_data (filter_column, other_column) INCLUDE (column1, column2);

Mandatory Predicates:

SQL
-- Ensure that all queries accessing `sensitive_data` must use the `idx_sensitive_data_filter` index
ALTER TABLE sensitive_data ADD CONSTRAINT c_mandatory_predicate CHECK ((filter_column = 'some_value') AND (other_column IN ('value1', 'value2')));

Cost Gates:

Configure database parameters to enforce cost limits:
SQL
-- Example: Setting a maximum query execution time in PostgreSQL
SET max_query_duration = 500;  -- Maximum query duration in milliseconds

-- Example: Configuring cost-based optimization in MySQL
SET optimizer_cost_based_join_enabled = ON;

Platform-Level Monitoring:

Use database monitoring tools to enforce and audit these constraints.
SQL
-- Example: Monitor query performance in PostgreSQL
SELECT * FROM pg_stat_statements WHERE query ~ 'sensitive_data';

#### Closing Summary

By implementing a combination of efficient indexes, mandatory predicates, cost gates, and platform-level enforcement mechanisms, we can ensure that every read on the sensitive table is provably bounded in cost. This approach not only meets regulatory requirements but also enhances overall database performance by guiding query execution to use optimal plans.

#### Common Mistakes/Follow-Ups

Mistake: Failing to consider partial indexes and their limitations.

#### Follow-Up
Discuss how partial indexes can be more effective for certain data subsets.

Mistake: Overlooking cost-based optimization parameters.

#### Follow-Up
Explain the importance of setting these parameters correctly to enforce bounded costs.

**Comparison:**

| Index Type | Columns            | Description                                      |
|------------|--------------------|--------------------------------------------------|
| Primary Key | id                 | Enforces unique constraints and fast access      |
| Covering   | filter_column, other_column | Includes necessary columns to avoid I/O          |
| Partial    | filter_column      | Limits data subset for more efficient indexing  |

**Code:**

```sql
-- Example SQL code for creating and modifying indexes
CREATE INDEX idx_sensitive_data_id ON sensitive_data (id);
ALTER TABLE sensitive_data ADD CONSTRAINT c_mandatory_predicate CHECK ((filter_column = 'some_value') AND (other_column IN ('value1', 'value2')));
SET max_query_duration = 500;
SELECT * FROM pg_stat_statements WHERE query ~ 'sensitive_data';
```

**Example input / output:**

```text
Input: SELECT * FROM sensitive_data WHERE filter_column = 'some_value';
Output: Query executed efficiently using idx_sensitive_data_filter index.
Input: SELECT * FROM sensitive_data;
Output: Query failed due to max_query_duration limit.
```

**Data:**

```json
{
    "index": {
        "id": "idx_sensitive_data_id",
        "columns": ["id"]
    },
    "predicate": {
        "constraint": "c_mandatory_predicate",
        "check": "(filter_column = 'some_value') AND (other_column IN ('value1', 'value2'))"
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Query Receives MandatoryPredicate
6. Is Predicate Satisfied?
7. Use Covered Index for Query
8. Exceed Cost Limit?
9. Fail Query Execution
10. Execute Query Efficiently UsingIndex

