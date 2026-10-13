---
type: "practice"
difficulty: "Intermediate"
---

# A producer proposes adding a nullable column. Is that a breaking change? Why or why not?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Contract Practice](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Data_Contract_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In the context of data engineering, data contracts define the structure and format of data inputs and outputs within a system. A producer proposing to add a nullable column raises questions about whether this change is considered a breaking change. This decision hinges on understanding how nullable columns interact with existing systems and contracts.

#### Triage

The question asks if adding a nullable column in a data contract constitutes a breaking change. The interviewer is probing your understanding of the implications of such changes, particularly how they affect downstream processes that rely on consistent schemas. We need to consider the current schema, potential impacts on existing systems, and the broader context of the data pipeline.

#### Assumptions
Current Schema: We are working with a well-defined schema where columns have specific types (e.g., INT, VARCHAR).

#### Cons
umer Systems: Existing consumers or processors that depend on the current schema.
Versioning Strategy: The system has some form of version control in place for its data contracts.

#### First-Principles Solution
Schema Definition:
A nullable column allows the inclusion of NULL values, meaning it can be empty or contain a value.
Impact on Consumers:

#### Cons
umers expecting non-nullable columns must handle the possibility of nulls, which could lead to errors if not accounted for.
Schema Evolution Strategy:
Adding a nullable column typically does not immediately break existing consumers since they can simply ignore or handle NULL values appropriately.
Step-by-Step Reasoning:
Initial Schema: Consider a simple schema where all columns are non-nullable:
SQL
CREATE TABLE example (
    id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    value DECIMAL NOT NULL
);
Proposed Change: Adding a nullable column to this schema changes it as follows:
SQL
ALTER TABLE example ADD COLUMN new_column VARCHAR(100);
Impact Analysis:
Non-Nullable Consumers: These systems would need to be updated to handle the possibility of nulls in new_column.
Nullable Consumers: Systems that are designed to handle nullable columns can continue functioning without changes.
Schema Versioning:
To manage this change, a new version of the data contract could be introduced:
JSON
"version": 2,
"schema": {
    "id": "INT",
    "name": "VARCHAR(255)",
    "value": "DECIMAL",
    "new_column": "VARCHAR(100) NULLABLE"
}
This allows for a smooth transition where old consumers can continue to operate with the previous schema, while new ones adopt the updated version.

#### Closing Summary

Adding a nullable column is generally not considered a breaking change because it does not immediately invalidate existing data or processes. However, it requires careful consideration of how downstream systems will handle null values and whether they need updates. The solution involves maintaining backward compatibility through schema evolution strategies like introducing new versions or handling NULL appropriately.

#### Common Mistakes/Follow-Ups
Failing to Handle NULLs: A common mistake is not updating consumers to correctly handle the possibility of null values.
Uncontrolled Schema Evolution: Rapid changes without proper versioning can lead to compatibility issues and system instability.
Ignoring Existing Systems: Failing to assess the impact on existing systems that depend heavily on non-nullable schemas.

**Comparison:**

| Column | Type       | Nullable |
|--------|------------|----------|
| id     | INT        | NO       |
| name   | VARCHAR    | NO       |
| value  | DECIMAL    | NO       |

**Code:**

```sql
-- Initial schema definition
CREATE TABLE example (
    id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    value DECIMAL NOT NULL
);

-- Schema evolution with nullable column addition
ALTER TABLE example ADD COLUMN new_column VARCHAR(100);
```

**Example input / output:**

```

Initial Schema (Version 1):

SQL
Copy
CREATE TABLE example (
    id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    value DECIMAL NOT NULL
);

Updated Schema (Version 2):

SQL
Copy
ALTER TABLE example ADD COLUMN new_column VARCHAR(100);
```

**Data:**

```json
{
  "version": 1,
  "schema": {
      "id": "INT",
      "name": "VARCHAR(255)",
      "value": "DECIMAL"
  }
}
```

**Flow:**

1. No
2. Yes
3. Initial Schema
4. Does the consumer handleNULLs?
5. Update consumers to handleNULLs
6. Schema is compatible, noaction needed
7. New schema versionintroduced

