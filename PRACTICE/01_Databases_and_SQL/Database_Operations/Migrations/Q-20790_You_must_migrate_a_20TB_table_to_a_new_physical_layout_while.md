---
type: "practice"
difficulty: "Expert"
---

# You must migrate a 20TB table to a new physical layout while it serves 200k QPS. Compare a logical-replication cutover, a shadow-table swap, and a blue-green DB approach.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrating to Partitioned Tables](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Migrating_to_Partitioned_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a 20TB table with heavy read/write operations (200k QPS) to a new physical layout, we need to carefully consider three approaches: logical-replication cutover, shadow-table swap, and blue-green database deployment. Each method has its trade-offs in terms of downtime, complexity, performance impact, and data consistency.

#### Triage

The primary goal is to minimize the duration and risk of the migration while ensuring data integrity. The interviewer is probing your understanding of each approach's pros and cons and how they can be applied to this specific scenario. We'll assume that no partial migrations are feasible due to the table size, and all changes must be done atomically.

Logical-Replication Cutover

This method involves using a logical replication stream to gradually migrate data from the old schema to the new one over time. Once the migration is complete, we can cut over to the new schema.

#### Assumptions
The database supports logical replication.
There's sufficient bandwidth for continuous data synchronization without affecting performance.
The application and queries are compatible with both schemas during the transition period.
Step-by-Step Solution
Setup Logical Replication: Configure a logical replication stream from the old table to the new schema.
Incremental Migration: Gradually migrate data by inserting into the new table while ensuring transactions are consistent.
Cutover: Once data migration is complete, update application configurations and redirect traffic to the new schema.
Shadow-Table Swap

This approach involves creating a shadow copy of the old table in the new schema, populating it, and then swapping the tables.

#### Assumptions
The database supports renaming or switching table names.
There's enough temporary storage for both the old and new tables during migration.
Applications can handle reading from two concurrent schemas temporarily.
Step-by-Step Solution
Create Shadow Table: Create an identical structure of the new schema in a temporary location (e.g., same database, different schema).
Populate Shadow Table: Insert data into the shadow table while ensuring consistency.
Swap Tables: Rename or swap the old and new tables atomically.
Blue-Green Deployment

This method involves running two versions of the system side by side: a current live version (blue) and an updated version (green). Once green is verified, traffic is switched over.

#### Assumptions
The database supports running multiple instances or schemas.
There's enough resources to support both systems simultaneously.
Applications can handle reading from two concurrent schemas during the transition period.
Step-by-Step Solution
Setup Green Environment: Create a new environment with the updated schema and populate it.
Verify Green: Test the green version thoroughly in parallel with the blue system.
Switch Traffic: Once verified, switch all traffic to the green environment.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The choice between these methods depends on the specific requirements of downtime tolerance, data consistency, and performance impact. For a 20TB table with high QPS, the shadow-table swap or blue-green approach might be more suitable due to their ability to handle heavy load without significant disruption.

#### Common Mistakes

Overlooking the need for extensive testing in blue-green deployments.
Underestimating the data migration time in logical replication cutover.
Failing to consider storage and performance impacts during shadow-table swap.
Self-Check

The solution covers all aspects of the problem, ensuring both correctness and completeness.

**Comparison:**

| Method            | Downtime          | Complexity        | Performance Impact | Data Consistency |
|-------------------|------------------|------------------|--------------------|------------------|
| Logical Replication Cutover | Minimal (hours)  | High             | Low                | Yes              |
| Shadow-Table Swap     | None (parallel)   | Medium           | High               | Yes              |
| Blue-Green DB         | None (side-by-side)| High             | Low to Medium      | Yes              |

**Code:**

```sql
-- Example SQL for setting up a shadow table
CREATE TABLE new_table AS SELECT * FROM old_table;
INSERT INTO new_table SELECT * FROM old_table; -- Populate data incrementally

-- Example SQL for cutover in logical replication
SELECT * FROM pg_replication_slots WHERE slot_name = 'logical_slot';

-- Example SQL for swapping tables (assuming temporary table `shadow`)
ALTER TABLE old_table RENAME TO old_table_temp;
ALTER TABLE new_table RENAME TO old_table;
ALTER TABLE old_table_temp RENAME TO new_table;
```

**Example input / output:**

```text
**Logical Replication Cutover Example:**

- **Input**: Start logical replication, migrate data.
- **Output**: Data in new schema is consistent and complete. Update application configurations.

**Shadow Table Swap Example:**

- **Input**: Create shadow table, insert data into it.
- **Output**: Tables are swapped atomically. Application traffic redirected to green environment.

**Blue-Green Deployment Example:**

- **Input**: Setup green environment, populate with new schema.
- **Output**: Verify green version thoroughly. Switch all traffic to green.
```

**Data:**

```json
{
  "tableSize": "20TB",
  "qps": "200k"
}
```

**Flow:**

1. Logical Replication Cutover
2. Start Replication
3. Migrate Data
4. Cutover Application
5. Minimal Downtime
6. Shadow-Table Swap
7. Create Shadow Table
8. Populate Data
9. Swap Tables Atomically
10. No Downtime
11. Blue-Green Deployment
12. Setup Green Env.
13. Populate New Schema
14. Test & Verify Green
15. Switch Traffic to Green
16. High Complexity, HighPerformance Impact

