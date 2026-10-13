---
type: "practice"
difficulty: "Expert"
---

# How do you tune the OLTP source for CDC — logical decoding slots, WAL retention, replica identity — without letting an unconsumed slot fill the disk?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Feeding OLAP from OLTP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Feeding_OLAP_from_OLTP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of tuning the OLTP (Online Transaction Processing) database for CDC (Change Data Capture)—specifically focusing on logical decoding slots, WAL retention, and replica identity—without letting an unconsumed slot fill the disk, we need to understand the core concepts involved and make strategic decisions. This solution requires a balance between minimizing data loss risk and optimizing resource utilization.

#### Triaging the Question

The interviewer is probing your understanding of how CDC works in OLTP databases, particularly focusing on logical decoding slots, Write-Ahead Log (WAL) retention policies, and replica identity constraints. The key challenge is to ensure that these configurations are set optimally without overwhelming disk space.

#### First-Principles Solution
Logical Decoding Slots

Logical decoding slots are essential for CDC because they allow the database to capture changes in a logical manner rather than relying on physical changes. Each slot acts as an independent stream of change data, which can be consumed by different applications or processes.

Slot Management: Ensure that you have enough slots to handle the expected volume of changes. For example, if your application expects 100 transactions per second, you might need a few slots to ensure no single slot becomes a bottleneck.
Replica Identity: When using logical decoding, replica identity constraints should be carefully managed to avoid data conflicts. Ensure that each slot is configured with appropriate constraints.
Write-Ahead Log (WAL) Retention

WAL retention policies determine how long the database retains transaction log entries before they are archived or deleted.

Retention Policy: Set a reasonable WAL retention policy based on your workload and recovery needs. For example, if you need fast recovery, you might want to retain logs for 24 hours. If disk space is a concern, you can set it lower.
Archiving: Configure the database to archive old log entries once they are no longer needed, such as after a certain period.
Replica Identity

Replica identity ensures that changes are applied consistently across all replicas of the database.

#### Cons
istency: Ensure that replica identity is configured correctly to avoid data conflicts and ensure consistency.
Synchronization: Monitor the synchronization state of your replicas to ensure they are up-to-date and free from inconsistencies.

#### Implementation
Example

Let's consider a scenario where we have an OLTP system with 500 transactions per second and we want to implement CDC using logical decoding:

Logical Decoding Slots:

SQL
-- Create slots for logical decoding
CREATE TABLE slot_table;
SELECT * FROM pg_create_logical_replication_slot('slot_name', 'decoding_plugin');

WAL Retention:

SQL
-- Set WAL retention policy to 1 day (86400 seconds)
ALTER SYSTEM SET wal_retention_policy = 'duration';
ALTER SYSTEM SET wal_retention_duration = '1d';

Replica Identity:

SQL
-- Ensure replica identity is configured correctly
CREATE TABLE primary_table (
    id SERIAL PRIMARY KEY,
    data TEXT NOT NULL,

#### CONS
TRAINT ri_unique_data UNIQUE (data)
);

SELECT * FROM pg_create_logical_replication_slot('slot_name', 'decoding_plugin');

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, the key is to configure logical decoding slots, WAL retention policies, and replica identity constraints in a way that maximizes efficiency without risking data loss or overloading disk space. You must carefully manage these settings based on your specific workload and recovery requirements.

#### Common mistakes
include:

Not having enough slots to handle the volume of changes.
Setting an overly aggressive WAL retention policy that fills up disk space.
Failing to properly configure replica identity constraints, leading to data inconsistencies.

#### Follow-ups
might involve discussing how to monitor and adjust these configurations dynamically based on real-time metrics or how to implement automated failover and recovery strategies.

**Code:**

```sql
-- Example SQL for creating logical decoding slots
CREATE TABLE slot_table;
SELECT * FROM pg_create_logical_replication_slot('slot_name', 'decoding_plugin');

-- Setting WAL retention policy
ALTER SYSTEM SET wal_retention_policy = 'duration';
ALTER SYSTEM SET wal_retention_duration = '1d';

-- Ensuring replica identity is configured correctly
CREATE TABLE primary_table (
    id SERIAL PRIMARY KEY,
    data TEXT NOT NULL,
    CONSTRAINT ri_unique_data UNIQUE (data)
);
SELECT * FROM pg_create_logical_replication_slot('slot_name', 'decoding_plugin');
```

