---
type: "practice"
difficulty: "Deep"
---

# Explain how the transaction log works internally (VLFs, log truncation, checkpoints), and what causes a log file to grow uncontrollably.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Read Committed Locking Anomalies](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL Server, the transaction log is a critical component that records all transactions on the database in sequential order. This record-keeping ensures data integrity by maintaining a history of changes and enabling point-in-time recovery. At a high level, the log consists of Virtual Log Files (VLFs) and operates through a process involving checkpoints and log truncation. However, under certain conditions, the log file can grow uncontrollably, which is a common issue that needs careful management.

The transaction log works by tracking every write operation on the database. Each transaction is written to the log in a sequential manner before being applied to the data files (MDFs). The transaction log uses VLFs as its basic unit of organization and growth, ensuring that changes are recorded persistently even during system failures. Log truncation is necessary to reclaim space once the transaction log has completed processing a transaction.

**Comparison:**

| VLF ID | Start Position  | End Position    |
|--------|-----------------|----------------|
| 1      | ...             | ...             |
| 2      | ...             | ...             |
| ...    | ...             | ...             |

**Code:**

```text
-- Example SQL to identify VLFs
DBCC LOGINFO (YourDatabaseName);
```

**Flow:**

1. Yes
2. No
3. Transaction Occurs
4. Check for Log Space
5. Log Writes to VLF
6. Log Truncation Checkpoint
7. Split VLF to Free Up Space
8. Checkpoints and LogTruncation Process
9. Truncate Unnecessary VLFs

