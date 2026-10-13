---
type: "practice"
difficulty: "Expert"
---

# A migration must repartition an existing 10TB partitioned table from range to hash partitioning. Design the phased cutover including how existing FKs and queries are handled.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrating to Partitioned Tables](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Migrating_to_Partitioned_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To successfully repartition a large, existing 10TB partitioned table from range to hash partitioning while maintaining foreign key (FK) relationships and ensuring query compatibility during a cutover, we'll design a phased migration strategy. This involves preparing the schema, handling FK constraints, partitioning data, and finally switching over services.

#### Triaging the Question

The core of this question is about designing a robust database migration process that minimizes downtime and ensures data integrity and query performance. The key aspects are:

Handling Foreign Key Constraints: Ensuring that existing tables with foreign keys (FKs) to the target table can still function during the transition.
Partitioning Data: Efficiently moving large volumes of data from range to hash partitions without causing service disruption.
Query Handling: Maintaining query performance and ensuring that application logic remains unchanged during migration.

#### Assumptions
The database is a relational system with support for partitioned tables and foreign keys.
FK relationships exist in multiple tables referencing the target table.
Existing queries leverage these partitions effectively.
Step-by-Step Solution

Preparation Phase

Schema Preparation:
SQL
-- Example schema changes
CREATE TABLE target_table_hash (
  id BIGINT PRIMARY KEY,
  /* other columns */
  PARTITION BY HASH(id);
);

ALTER TABLE existing_table_r RANGE TO target_table_hash;
FK Relationship Management: Ensure that FKs to the old table are temporarily removed or managed during the transition.
SQL
-- Temporarily remove or disable FK constraints
ALTER TABLE referencing_table DROP FOREIGN KEY fk_name;

Data Migration Phase

Incremental Data Load: Use a phased approach to incrementally move data from range partitions to hash partitions, ensuring minimal impact on running services.
SQL
-- Example: Incremental load of 10% at a time
INSERT INTO target_table_hash SELECT * FROM existing_table_r LIMIT 100000;
Data Validation: Run validation checks to ensure data consistency and accuracy after each batch.
SQL
-- Validate data integrity
SELECT COUNT(*) AS total, SUM(some_column) AS sum FROM target_table_hash GROUP BY hash_partition;

Service Update Phase

Modify Application Logic: Adjust application queries to use the new partitioning strategy without breaking existing functionality.
SQL
-- Example query adjustment
SELECT * FROM target_table_hash WHERE id BETWEEN 100 AND 200;
Update FK Constraints: Once all data has been migrated, re-enable or recreate FK constraints to ensure referential integrity.
SQL
-- Re-create FKs after migration is complete
ALTER TABLE referencing_table ADD CONSTRAINT fk_name FOREIGN KEY (fk_column) REFERENCES target_table_hash(id);

Final Cutover

Full Service Switch: Once the migration and validation are complete, switch all services to use the new hash-partitioned table.
SQL
-- Final switch of services
ALTER TABLE existing_table_r RENAME TO target_table_old;
ALTER TABLE target_table_hash RENAME TO target_table;

#### Closing Summary
+ Common Mistakes/Follow-Ups

Summary: The key to a successful migration is a phased approach that minimizes downtime, ensures data integrity, and maintains query performance. By carefully planning the schema changes, incremental data loads, and service updates, we can ensure a smooth transition.

#### Common Mistakes

Failing to handle FK constraints properly during the transition.
Overlooking validation checks leading to data inconsistencies.
Not thoroughly testing queries after migration, resulting in performance issues or logical errors.

#### Follow-Ups
How would you manage this process if the target table was sharded across multiple databases?
What measures would you take to ensure data consistency during the migration of large volumes?

**Code:**

```sql
-- Example schema changes
CREATE TABLE target_table_hash (
  id BIGINT PRIMARY KEY,
  /* other columns */
) PARTITION BY HASH(id);

ALTER TABLE existing_table_r RANGE TO target_table_hash;

-- Example: Incremental data load
INSERT INTO target_table_hash SELECT * FROM existing_table_r LIMIT 100000;

-- Validate data integrity
SELECT COUNT(*) AS total, SUM(some_column) AS sum FROM target_table_hash GROUP BY hash_partition;

-- Final switch of services
ALTER TABLE existing_table_r RENAME TO target_table_old;
ALTER TABLE target_table_hash RENAME TO target_table;
```

**Flow:**

1. Preparation
2. Incremental Data Load
3. Data Validation
4. Service Update
5. Final Cutover

