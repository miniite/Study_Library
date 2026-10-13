---
type: "practice"
difficulty: "Deep"
---

# What is a 'write-write conflict' under snapshot isolation and how does the database detect and resolve it at commit time?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lost Updates and Write Skew](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Lost_Updates_and_Write_Skew.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Snapshot Isolation, write-write conflicts occur when two transactions concurrently update the same row, but one of them sees an old value of that row from a snapshot taken before the other transaction started. This conflict needs to be detected and resolved at commit time to ensure consistency.

#### Triage

The question probes your understanding of how Snapshot Isolation handles concurrent writes, specifically focusing on write-write conflicts. The key points are:

Conflict Detection: How the database identifies conflicting updates.
Resolution Mechanism: What happens when a conflict is detected during commit.

#### First-Principles Solution
Conflict Detection

Under Snapshot Isolation (SI), each transaction sees a snapshot of the database as it existed at the start of the transaction. This means that if one transaction updates a row, and another transaction reads the same row later but sees the old value before the update, they are in a write-write conflict situation.

To detect such conflicts:

When a transaction attempts to commit its changes, the database checks whether any other transactions have modified the rows being updated by this transaction. If so, a conflict is detected.
Resolution Mechanism

When a conflict is detected at commit time, the database has several strategies to resolve it:

Row Versioning: Each row version (snapshot) is stored with its transaction ID and timestamp. When committing changes, the database checks if any newer versions of the row exist in other transactions' snapshots.

#### Cons
istency Checks: The database performs consistency checks by comparing the committed values against the snapshot values read at the start of the transaction.
Step-by-Step Solution

Transaction Start:

Transaction T1 starts and reads a row R with value V1 from Snapshot S1.

Conflict Scenario:

Meanwhile, Transaction T2 modifies R to new value V2 and is committed, creating Snapshot S2.
Later, Transaction T3 reads the same row R using Snapshot S1, sees V1.

Commit Time Detection:

When T3 attempts to commit its changes to update R from V1 to V3, the database detects that a newer snapshot (S2) exists with value V2.
A write-write conflict is identified because both T3 and T2 have updated the same row.

Conflict Resolution:

The database decides how to handle this conflict. Common strategies include:
Deny Commit: Reject T3's commit if it conflicts with a newer snapshot (S2).
Retry: Allow T3 to re-read and re-commit, possibly with an updated value.
Conflict Logging: Log the conflict for debugging or auditing purposes.

#### Closing Summary

Under Snapshot Isolation, write-write conflicts are detected at commit time by comparing the transaction's snapshot against newer versions of the rows. The database resolves these conflicts through various mechanisms to ensure data consistency and integrity. Common strategies include denying commits, retrying transactions, or logging conflicts.

#### Common Mistakes/Follow-Ups
Failing to Consider Snapshot Versions: Often, developers assume that a transaction sees the latest version of all rows, neglecting how snapshots work.
Ignoring Transaction Timestamps: Conflicts are not detected by timestamps alone but require comparing snapshot versions.
Not Handling Retry Logic: Understanding when and how to retry transactions is crucial for robust database management.

**Comparison:**

| Transaction | Row Value  |
|-------------|------------|
| T1          | V1         |
| T2 (Snapshot)| V2         |
| T3 (Snapshot)| V1         |

**Code:**

```sql
-- Example SQL logic for detecting conflicts during commit
BEGIN;
SELECT row_value FROM table WHERE id = 1 FOR UPDATE;
UPDATE table SET value = new_value WHERE id = 1 AND row_value = old_value;
```

**Example input / output:**

```text
Input:
- Transaction T1 starts and reads row R with value V1.
- Transaction T2 modifies R to V2 and commits, creating snapshot S2.
- Transaction T3 reads the same row using Snapshot S1 (shows V1).

Output:
- At commit time for T3: Conflict detected due to newer snapshot S2 exists.
- Resolution strategy applied is "Deny Commit".
```

**Data:**

```json
{
  "transactionSnapshots": {
    "T1": {"rowR": "V1"},
    "T2": {"rowR": "V2"},
    "T3": {"rowR": "V1"}
  },
  "conflictDetected": true,
  "resolutionStrategy": "Deny Commit"
}
```

**Flow:**

1. Transaction Starts
2. Reads row R with value V1
3. Conflicts detected when tryingto commit
4. Deny Commit
5. Conflict logged
6. Retry Transaction

