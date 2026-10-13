---
type: "practice"
difficulty: "Expert"
---

# Your team wants to switch a MySQL table from InnoDB to MyRocks to save disk. Walk through the migration risks: online DDL, secondary index behavior, transaction isolation differences, and how you'd validate parity before cutover.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ B-tree Concurrency and Latches](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_Concurrency_and_Latches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating a MySQL table from InnoDB to MyRocks, understanding the nuances of B-tree and Log-Structured Merge (LSM) tree internals is crucial. This migration involves several key areas that need careful consideration to ensure minimal downtime and data integrity. The main risks include online DDL capabilities, secondary index behavior, transaction isolation differences, and validating parity before a full cutover.

Firstly, let's outline the core migration steps:

Online DDL: InnoDB supports online DDL operations like ALTER TABLE, which can add or drop columns without locking the table for long periods. MyRocks also supports similar features but with some differences.
Secondary Indexes: Both engines have secondary indexes, but their handling and performance characteristics differ significantly.
Transaction Isolation Levels: These are crucial for maintaining data consistency during migrations.
Validation Before Cutover: Ensuring the data in both engines is identical before making the final switch.

Now, let's dive into each risk area:

Online DDL

InnoDB uses a transactional approach to handle ALTER TABLE, ensuring that changes can be made without locking the entire table for extended periods. MyRocks also supports online DDL but may behave differently due to its internal storage structure. Specifically:

InnoDB: Uses a combination of clustered and secondary indexes.
MyRocks: Uses an LSM tree, which allows for efficient updates while maintaining a level of data integrity.

During the migration, you need to ensure that any ongoing DDL operations in InnoDB are completed or paused before switching over. You also have to check if MyRocks can handle these same operations without significant performance degradation.

Secondary Indexes

Secondary indexes in both engines operate differently:

InnoDB: Uses a clustered index (primary key) and multiple secondary B-trees.
MyRocks: Uses an LSM tree where each index is stored as a level of the tree, with the main data being stored in a single file.

This means that during the migration, you need to ensure all secondary indexes are properly maintained. InnoDB might require a full re-indexing process, which could be resource-intensive. MyRocks, on the other hand, can handle these changes more gracefully due to its write-optimized nature.

Transaction Isolation Levels

Transaction isolation levels in MySQL are critical for ensuring data consistency and preventing issues like dirty reads, non-repeatable reads, and phantom reads:

InnoDB: Supports Serializable, Repeatable Read, and Read Committed.
MyRocks: Primarily supports Snapshot Isolation (SI), which is a form of Repeatable Read.

During the migration, you need to ensure that any existing transactions using InnoDB isolation levels are compatible with MyRocks. This might require adjusting transactional workloads or handling potential inconsistencies during the switch.

Validation Before Cutover

Before making the full cutover, it's essential to validate that the data in both engines is identical:

Full Table Scan: Perform a full table scan on both InnoDB and MyRocks to compare all rows.
Checksums: Use checksums or other integrity checks to ensure data consistency.
Index Verification: Verify that secondary indexes are correctly built and consistent.

To validate these, you can use tools like pt-table-checksum from Percona Toolkit for database comparisons. You might also set up a test environment where both engines run side by side for a period to catch any discrepancies early.

#### Closing Summary

By carefully considering the differences in online DDL support, secondary index behavior, and transaction isolation levels, you can mitigate most risks during the migration from InnoDB to MyRocks. Ensuring thorough validation before the final cutover is critical to maintaining data integrity and performance.

#### Common Mistakes/Follow-Ups
Ignoring Online DDL Differences: Failing to properly manage ongoing DDL operations can lead to unexpected downtime or data inconsistencies.
Inadequate Index Management: Not handling secondary indexes correctly during the migration can result in suboptimal query performance on MyRocks.
Insufficient Transaction Isolation Testing: Failing to test transactional workloads thoroughly may introduce bugs related to isolation levels.
Skipping Pre-Migration Validation: Rushing into the cutover without proper validation can lead to data discrepancies and service disruptions.

**Comparison:**

| InnoDB | MyRocks |
|--------|---------|
| B-tree | LSM tree |
| Clustered index | Single file storage with multiple levels of indexes |
| Support for Online DDL | Snapshot Isolation (SI) |

**Code:**

```sql
-- Example of a simple ALTER TABLE operation in InnoDB
ALTER TABLE mytable ADD COLUMN new_column INT;

-- Example of a MyRocks DDL operation
ALTER TABLE mytable ADD COLUMN new_column INT;
```

**Example input / output:**

```text
Input:
- InnoDB table with existing data and indexes.
- MyRocks table structure defined.

Output:
1. Validate primary key and index consistency across both engines using checksums.
2. Perform a full table scan for row-level comparison.
3. Ensure transaction isolation levels are compatible during the migration window.
```

**Data:**

```json
{
  "table_name": "mytable",
  "columns": [
    { "name": "id", "type": "INT" },
    { "name": "name", "type": "VARCHAR(255)" }
  ],
  "indexes": [
    { "name": "idx_id", "type": "B-Tree", "columns": ["id"] },
    { "name": "idx_name", "type": "B-Tree", "columns": ["name"] }
  ]
}
```

**Flow:**

1. @startuml
2. title Migration from InnoDB to MyRocks - Flow
3. node InnoDB {
4. node Table Operations {
5. alter table add column
6. insert delete update
7. }
8. }
9. node MyRocks {
10. node Table Operations {
11. alter table add column
12. insert delete update
13. }
14. }
15. node Validation {
16. validate indexes and data consistency
17. perform full table scan
18. run transaction isolation tests
19. }
20. start -> InnoDB: Validate pre-migration conditions
21. InnoDB: Table Operations
22. InnoDB -> MyRocks: Migrate schema changes
23. MyRocks: Table Operations
24. MyRocks -> Validation: Verify data integrity
25. Validation -> InnoDB: Ensure compatibility with existing transactions
26. InnoDB -> MyRocks: Perform cutover if validated
27. stop
28. @enduml

