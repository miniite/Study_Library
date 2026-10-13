---
type: "practice"
difficulty: "Expert"
---

# How would you back up and restore a multi-terabyte database with an RPO of 5 minutes and RTO of 1 hour? Explain WAL archiving, PITR, and how you'd actually test the restore.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Replica Staleness](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To back up and restore a multi-terabyte database with an Recovery Point Objective (RPO) of 5 minutes and a Recovery Time Objective (RTO) of 1 hour, we need to leverage WAL archiving and Point-In-Time Recovery (PITR). This approach ensures that the system can recover data quickly after a failure while maintaining minimal downtime. Let's break down the solution step by step.

Firstly, it’s important to understand the key components:

WAL Archiving: Writes-Ahead Logging (WAL) is used for transaction log archiving. It records every modification made to the database in sequence.
PITR: Point-In-Time Recovery allows you to recover your database state at any point in time by using archived WAL segments.

#### Triaging the Question

The interviewer is probing on:

Your understanding of advanced backup strategies for large databases.
Your ability to implement robust recovery mechanisms that meet specific RPO and RTO requirements.
Your practical experience with PostgreSQL, which often supports these features (other DBMS may have similar concepts).

#### First-Principles Solution
1. WAL Archiving Setup
Enable WAL archiving on the database server to ensure transaction logs are stored separately from the main data files.
Configure the archive_mode and archive_command settings in PostgreSQL’s configuration file (postgresql.conf):
PLAINTEXT
archive_mode = on
archive_command = 'test ! -f /path/to/backup/%p && cp %p /path/to/backup/%f'
Set up a secondary storage location to hold the archived WAL segments, ensuring it has enough space.
2. Retention Period
Decide on an appropriate retention period for archived WAL segments based on your RPO of 5 minutes. A common practice is to retain them for several hours.
Use archive_timeout to automatically delete old WAL segments:
PLAINTEXT
archive_timeout = 300s
3. PITR Configuration
Configure the primary database to enable PITR by setting up a recovery configuration file (recovery.conf) with the necessary parameters:
PLAINTEXT
restore_command = 'cp /path/to/backup/%f %p'
Ensure that you can initiate a recovery process from any point within your retention period.
4. Testing the Restore
Perform regular tests to verify that the backup and recovery process works as expected.
Simulate a failure scenario, such as shutting down the primary database and starting it with recovery_target_time set to a specific timestamp:
PLAINTEXT
recovery_target_time = '2023-10-05 14:00:00+00'

#### Closing Summary

By setting up WAL archiving and configuring PITR, you can ensure that your database is backed up with an RPO of 5 minutes and the system can recover within a 1-hour RTO. Regular testing will help you validate the effectiveness of these backup strategies.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to configure archive_command properly, leading to incomplete archiving.

#### Follow-Up
How would you handle recovery in a distributed database environment? What about data consistency during and after restoration?

