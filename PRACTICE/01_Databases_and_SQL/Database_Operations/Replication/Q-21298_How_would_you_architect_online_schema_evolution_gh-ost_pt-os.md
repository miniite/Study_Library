---
type: "practice"
difficulty: "Expert"
---

# How would you architect online schema evolution (gh-ost / pt-osc) across a replicated topology so a DDL on the master never stalls the whole apply pipeline?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes and Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Schema_Changes_and_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect online schema evolution in a replicated database topology, we need to ensure that changes made to the master database do not disrupt the replication process or cause downtime on the slave databases. The key concept here is using tools like gh-ost (GoldenGate Online Schema Transformation) and pt-osc (Percona Toolkit Online DDL). These tools are designed to perform schema modifications in a non-blocking manner, allowing for minimal impact during database maintenance.

#### Triage

The question asks how to ensure that a Data Definition Language (DDL) change on the master does not stall the entire replication pipeline. This involves understanding how gh-ost or pt-osc works under the hood and leveraging their capabilities to minimize disruption. We'll assume we're working with a MySQL/MariaDB environment, as these tools are commonly used for this purpose.

#### First Principles Solution
Understand the Replication Process: In a master-slave replication setup, changes on the master are recorded in binary logs, which are then applied to the slave via the I/O thread and SQL thread.
Utilize gh-ost or pt-osc: These tools insert temporary tables with data from the production table, perform DDL operations, and then swap the temporary and original tables to minimize downtime.
Step-by-Step Solution

Identify the Table to Evolve:

SQL
SELECT * FROM your_table LIMIT 0;

Create a Temporary Table with a Unique Identifier:

SQL
CREATE TABLE temp_your_table LIKE your_table;
ALTER TABLE temp_your_table ADD COLUMN temp_unique_id INT PRIMARY KEY AUTO_INCREMENT;

Replicate the Data from the Original to the Temporary Table (using gh-ost):

SH
gh-ost --user=root --password=your_password --host=localhost \
       --rpo=60 --strategy=partial \
       --alter "ADD COLUMN new_column INT" \
       --table=your_table --tablespace=none

Perform the DDL on the Original Table:

SQL
ALTER TABLE your_table ADD COLUMN new_column INT;

Swap Tables:

SQL
RENAME TABLE your_table TO old_your_table, temp_your_table TO your_table;
DROP TABLE old_your_table;

Verify and Test: Ensure that the schema change has been applied correctly to both master and slave.

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach minimizes downtime by allowing DDL operations to be performed in a way that does not block replication threads. Key considerations include ensuring proper error handling, monitoring the progress of gh-ost, and testing thoroughly before applying these changes in production environments.

#### Common mistakes
include:

Not understanding the limitations or failure modes of gh-ost/pt-osc.
Failing to monitor the replication lag post-schema change.
Neglecting to test non-trivial DDL operations exhaustively.

For follow-ups, one might discuss how these tools handle concurrent queries, transactional integrity during schema changes, and how they integrate with other database management tools.

**Code:**

```sql
-- Example SQL commands for gh-ost/pt-osc
CREATE TABLE temp_your_table LIKE your_table;
ALTER TABLE temp_your_table ADD COLUMN temp_unique_id INT PRIMARY KEY AUTO_INCREMENT;

INSERT INTO temp_your_table SELECT * FROM your_table;

ALTER TABLE your_table ADD COLUMN new_column INT;

RENAME TABLE your_table TO old_your_table, temp_your_table TO your_table;
DROP TABLE old_your_table;
```

**Flow:**

1. Master DDL
2. Binary Log
3. Slave I/O Thread
4. SQL Thread
5. Apply Schema Changes

