---
type: "practice"
difficulty: "Deep"
---

# How do you validate that your standby database is actually caught up and safe to promote during a disaster?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backup Verification](<../../../../01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Backup_Verification.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a standby database can be safely promoted in case of a disaster, validation steps are crucial to guarantee data consistency and integrity. This involves both procedural checks and technical validations at various levels—log sequence number (LSN) comparison, transaction logs, and application-level checks.

#### Triage

The question revolves around ensuring the standby database is fully synchronized and consistent with the primary database before a failover or promotion can occur. We need to confirm that all transactions have been applied correctly, no data loss has occurred, and the system state on both databases matches.

#### Assumptions
The standby database is using synchronous or asynchronous replication.
There are mechanisms in place for log shipping or real-time mirroring.
Logs or checkpoint files are used to track transactional changes.

#### First-Principles Solution

LSN Comparison and Transaction Logs:

Step 1: Check the latest LSN on both primary and standby databases.
Step 2: Ensure that all log records up to the highest LSN have been applied on the standby.
Step 3: Verify that no gaps exist in transaction logs between the two systems.

Application-Level Checks:

Step 1: Execute consistency checks using predefined queries or stored procedures.
Step 2: Compare key metrics and states, such as table counts, row versions, and any application-specific data integrity checks.

Manual Verification Steps:

Step 1: Manually run a few critical transactions to simulate the failover process.
Step 2: Verify that these transactions behave identically on both databases.

Automated Monitoring Tools:

Use database monitoring tools to ensure real-time synchronization and alerting mechanisms for any discrepancies.
Detailed Steps

LSN Comparison:

SQL
-- On the primary database
SELECT MAX(lsn) AS max_lsn FROM transaction_log WHERE state = 'applied';

-- On the standby database
SELECT MAX(lsn) AS max_lsn FROM transaction_log WHERE state = 'applied';

Transaction Log Verification:

Compare the log sequences to ensure they match.
Use tools like DBCC OPENTRAN or equivalent commands if necessary.

Application-Level Checks:

SQL
-- Example: Check table counts and row versions for consistency
SELECT COUNT(*) FROM main_table;

Manual Simulation:

Run a few transactions to simulate the failover process.
Ensure these transactions work identically on both databases.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes
Failing to verify LSNs, skipping application-level checks, or relying solely on monitoring tools without manual validation.

#### Follow-Ups
Discussing potential failure modes like network issues during log shipping and how to mitigate them. Also, exploring different replication strategies (synchronous vs. asynchronous) and their implications.

**Code:**

```sql
-- Example LSN comparison script
SELECT MAX(lsn) AS max_lsn FROM transaction_log WHERE state = 'applied';
```

**Flow:**

1. Primary DB LSN
2. Compare with
3. Standby DB LSN
4. Match Check
5. Transaction Logs Applied
6. No Gaps Found
7. Application-Level Checks
8. Consistent Results
9. Manual Simulation
10. Identical Behavior

