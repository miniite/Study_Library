---
type: "practice"
difficulty: "Deep"
---

# How do you safely add a NOT NULL column with a computed default to a large table on Postgres 11+ vs. older versions, and why did older versions rewrite the whole table?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in Lakehouse Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To safely add a NOT NULL column with a computed default to a large PostgreSQL 11+ table, you need to leverage the DEFAULT clause and ensure that no rows violate this constraint during the alteration process. The key difference between older versions (pre-PostgreSQL 11) and newer versions lies in how they handle the rewriting of the entire table.

#### Triaging the Question

The question is essentially about modifying a database schema to add a NOT NULL column with a default value, while ensuring data integrity and minimizing performance impact. The interviewer is probing your understanding of PostgreSQL's version-specific behaviors, particularly focusing on default values and table rewrites.

#### Assumptions
PostgreSQL Version: We are considering two scenarios: pre-PostgreSQL 11 (where tables might be rewritten) versus PostgreSQL 11+ (where the process is more efficient).
Column Type: Assume the new column type is such that it cannot contain NULL values.
Default Value: The default value for this column is computed and can be determined during insertion.

#### Solution
Step-by-Step Approach Using PostgreSQL 11+

In PostgreSQL 11+, you can add a NOT NULL column with a default value using the following steps:

SQL
-- Add the new column with NOT NULL constraint, but without data yet.
ALTER TABLE your_table ADD COLUMN new_column_name type DEFAULT computed_default_value;

-- Populate the table with the default value for existing rows.
UPDATE your_table SET new_column_name = computed_default_value WHERE new_column_name IS NULL;
Step-by-Step Approach in Pre-PostgreSQL 11 Versions

For pre-PostgreSQL 11 versions, you need to handle the addition and population of data more carefully:

SQL
-- Add the new column without a NOT NULL constraint.
ALTER TABLE your_table ADD COLUMN new_column_name type;

-- Populate the table with default values for existing rows.
UPDATE your_table SET new_column_name = computed_default_value WHERE new_column_name IS NULL;

-- Once all rows have valid values, add the NOT NULL constraint.
ALTER TABLE your_table ALTER COLUMN new_column_name SET NOT NULL;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, PostgreSQL 11+ allows a more streamlined approach by adding the NOT NULL constraint alongside the default value, reducing the risk of data integrity issues during column addition. Older versions require an additional step to handle the NOT NULL constraint post-population.

#### Common mistakes
include:

Not handling existing rows correctly (e.g., forgetting to update or ensuring no NULL values).
Adding a NOT NULL constraint without sufficient validation, leading to potential data loss.
Ignoring the version-specific behavior and assuming both behave similarly.

#### Follow-up
questions might explore edge cases like concurrent access during modifications, constraints on foreign keys, or performance considerations during large-scale operations.

**Comparison:**

| Scenario | PostgreSQL Version | Key Steps |
|----------|--------------------|-----------|
| PostgreSQL 11+ | Newer | Add column with default; populate data; add NOT NULL constraint. |
| Pre-PostgreSQL 11 | Older | Add column without default; populate data; add NOT NULL constraint after population. |

**Code:**

```sql
-- PostgreSQL 11+ approach
ALTER TABLE your_table ADD COLUMN new_column_name type DEFAULT computed_default_value;

UPDATE your_table SET new_column_name = computed_default_value WHERE new_column_name IS NULL;

-- Older versions approach
ALTER TABLE your_table ADD COLUMN new_column_name type;

UPDATE your_table SET new_column_name = computed_default_value WHERE new_column_name IS NULL;

ALTER TABLE your_table ALTER COLUMN new_column_name SET NOT NULL;
```

**Example input / output:**

```text
-- Initial state of the table
-- Existing data with some NULL values for a new column

your_table = [
  { existing_column1: 1, existing_column2: 'A' },
  { existing_column1: null, existing_column2: 'B' },
  { existing_column1: 3, existing_column2: null }
]

-- After adding the NOT NULL column with a default value in PostgreSQL 11+

your_table = [
  { existing_column1: 1, existing_column2: 'A', new_column_name: 42 },
  { existing_column1: null, existing_column2: 'B', new_column_name: 0 },
  { existing_column1: 3, existing_column2: null, new_column_name: -1 }
]

-- After setting the column to NOT NULL

your_table = [
  { existing_column1: 1, existing_column2: 'A', new_column_name: 42 },
  { existing_column1: null, existing_column2: 'B', new_column_name: 0 }, -- This row will fail due to constraint violation
  { existing_column1: 3, existing_column2: null, new_column_name: -1 }
]
```

**Data:**

```json
{
  "table_schema": {
    "name": "your_table",
    "columns": [
      { "name": "existing_column1", "type": "integer" },
      { "name": "existing_column2", "type": "text" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is PostgreSQL version >= 11?
5. Add column with DEFAULT andNOT NULL
6. Add column without DEFAULT;Populate data; Add NOT NULLconstraint
7. End
8. End

