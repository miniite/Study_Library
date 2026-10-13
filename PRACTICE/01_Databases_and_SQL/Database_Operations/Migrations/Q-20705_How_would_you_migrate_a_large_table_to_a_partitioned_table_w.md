---
type: "practice"
difficulty: "Expert"
---

# How would you migrate a large table to a partitioned table with minimal downtime? What's your cutover plan?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Migrating to Partitioned Tables](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Migrating_to_Partitioned_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To migrate a large table to a partitioned table with minimal downtime, we need a comprehensive strategy that involves several steps: data analysis, schema design, incremental migration, and cutover planning. This process ensures that the application can continue operating without significant disruption.

#### Triaging the Question

The interviewer is probing your understanding of database management practices, particularly in handling large datasets efficiently with minimal downtime during migrations. Key aspects include:

Data Analysis: Understanding the current schema and data distribution.
Schema Design: Crafting a partitioning strategy that aligns with business requirements.
Incremental Migration: A phased approach to minimize disruptions.
Cutover Plan: Smooth transition from old to new table structure.

#### Assumptions
The database is using PostgreSQL, which supports Common Table Expressions (CTEs) and partitioned tables natively.
The application can handle some degree of read-only or write-freeze time during migration phases.
The target schema design allows for incremental data migration without blocking the entire table.

#### First-Principles Solution
Step 1: Data Analysis

First, analyze the existing large table to understand its structure and content:

Columns: Identify key columns that will determine partitioning criteria (e.g., date, range).
Size: Estimate the total data volume and average row size.
Usage Patterns: Understand read/write patterns to optimize partition keys.
Step 2: Schema Design

Design a new partitioned table schema:

Partition Key Selection: Choose a column that can effectively distribute data across partitions (e.g., date or region).
Partitioning Type: Decide between range, list, or hash partitioning based on the nature of the data and query patterns.
Partition Columns: Add necessary columns to support new table structure.

#### Example
Schema:

SQL
CREATE TABLE new_table (
    id SERIAL PRIMARY KEY,
    region VARCHAR(50),
    value DECIMAL(12, 2),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) PARTITION BY RANGE (timestamp);
Step 3: Incremental Data Migration

Implement a phased migration strategy to minimize downtime:

Create Partitions: Dynamically create partitions based on the chosen partition key.
Data Transfer:
Use INSERT INTO ... SELECT statements to transfer data in batches.
Utilize CTEs for complex queries if needed.
Incremental Load: Gradually load data into new tables while keeping old tables accessible.

#### Example
Incremental Migration:

SQL
-- Create partitioned table with initial partitions
CREATE TABLE new_table_202301 PARTITION OF new_table FOR VALUES FROM ('2023-01-01') TO ('2023-02-01');
CREATE TABLE new_table_202302 PARTITION OF new_table FOR VALUES FROM ('2023-02-01') TO ('2023-03-01');

-- Transfer data from old to new table in batches
INSERT INTO new_table_202301 SELECT * FROM old_table WHERE timestamp >= '2023-01-01' AND timestamp < '2023-02-01';
Step 4: Cutover Plan

Develop a plan to transition the application from the old table to the new:

Application Configuration: Update application configurations to point to the new table.
Read-Only Phase: Temporarily switch to read-only mode for the old table.
Write-Freeze Phase: Freeze writes on the old table during final migration steps.
Final Migration: Complete any remaining data transfer and cleanup operations.

#### Example
Cutover Steps:

Update Application Configuration:

Change application database connection strings or configuration files to point to new_table.

Read-Only Mode:

SQL
ALTER TABLE old_table SET (ACCESSIBLE = FALSE);

Write-Freeze Mode:

SQL
ALTER TABLE old_table NO INHERIT new_table;

Final Migration:

Ensure all data is transferred and indexes are created on the new table.
Verify data consistency between old and new tables.

#### Closing Summary

By following a structured approach involving detailed analysis, thoughtful schema design, incremental data migration, and a well-planned cutover, we can ensure minimal downtime during the transition to partitioned tables. This strategy balances operational continuity with efficient data management practices.

#### Common Mistakes/Follow-Ups
Incomplete Data Transfer: Failing to transfer all relevant data in each phase.
Ignoring Performance Impacts: Not considering the performance implications of large-scale operations.
Overcomplicating Schema Design: Compromising on simplicity for overly complex partitioning strategies.

For follow-ups, consider discussing tools like pg_partman for automated partition management or exploring parallel migration techniques to further reduce downtime.

**Comparison:**

```
FLOWCHART · MERMAID
erDiagram
    OLD_TABLE ||--o{ NEW_TABLE : inherits from
    NEW_TABLE {
        id SERIAL PRIMARY KEY
        region VARCHAR(50)
        value DECIMAL(12, 2)
        timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    }
    NEW_TABLE_202301 ||-o{ NEW_TABLE : part of
    NEW_TABLE_202302 ||-o{ NEW_TABLE : part of

    NEW_TABLE_202301 {
        id SERIAL PRIMARY KEY
        region VARCHAR(50)
        value DECIMAL(12, 2)
        timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        constraint new_table_partition FOR VALUES FROM ('2023-01-01') TO ('2023-02-01')
    }

    NEW_TABLE_202302 {
        id SERIAL PRIMARY KEY
        region VARCHAR(50)
        value DECIMAL(12, 2)
        timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        constraint new_table_partition FOR VALUES FROM ('2023-02-01') TO ('2023-03-01')
    }
```

**Code:**

```sql
-- Example SQL commands for creating and migrating data

CREATE TABLE new_table (
    id SERIAL PRIMARY KEY,
    region VARCHAR(50),
    value DECIMAL(12, 2),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) PARTITION BY RANGE (timestamp);

CREATE TABLE new_table_202301 PARTITION OF new_table FOR VALUES FROM ('2023-01-01') TO ('2023-02-01');
CREATE TABLE new_table_202302 PARTITION OF new_table FOR VALUES FROM ('2023-02-01') TO ('2023-03-01');

INSERT INTO new_table_202301 SELECT * FROM old_table WHERE timestamp >= '2023-01-01' AND timestamp < '2023-02-01';

ALTER TABLE old_table SET (ACCESSIBLE = FALSE);
ALTER TABLE old_table NO INHERIT new_table;
```

**Example input / output:**

| Old Table Data (Sample) | New Table Migration Status |
|---|---|
| id: 1, region: US, value: 45.67, timestamp: '2023-01-15' | Migrated to new_table_202301 |
| id: 2, region: CA, value: 98.76, timestamp: '2023-02-20' | Migrated to new_table_202302 |

**Data:**

```yaml
# Example data schema and partitioning criteria
table: "old_table"
columns:
  - id: SERIAL PRIMARY KEY
  - region: VARCHAR(50)
  - value: DECIMAL(12, 2)
  - timestamp: TIMESTAMP DEFAULT CURRENT_TIMESTAMP

partitions:
  - name: new_table_202301
    range: "2023-01-01" TO "2023-02-01"
  - name: new_table_202302
    range: "2023-02-01" TO "2023-03-01"
```

**Flow:**

1. Analyze Existing Table
2. Design Partitioned Schema
3. Migrate Data Incrementally
4. Configure Application to Pointto New Tables
5. Finalize Migration and VerifyData Integrity

