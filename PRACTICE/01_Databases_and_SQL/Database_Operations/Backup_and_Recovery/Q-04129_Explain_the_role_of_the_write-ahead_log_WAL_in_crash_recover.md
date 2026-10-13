---
type: "practice"
difficulty: "Deep"
---

# Explain the role of the write-ahead log (WAL) in crash recovery and how it guarantees durability.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Consistent Backups and Restore Safety](<../../../../01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Consistent_Backups_and_Restore_Safety/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a database system experiences a crash, ensuring that all writes are properly recorded is critical to maintaining data integrity. The write-ahead log (WAL), also known as a transaction log or redo log, plays a crucial role in this process by providing a sequential record of all changes made to the database. By leveraging the WAL, the database can recover from a crash and ensure that no transactions are lost.

In essence, the WAL acts like a journal where each data modification is recorded before it is applied to the actual data files. This design ensures that even if the system crashes after writing the log but before updating the database files, the changes will still be applied upon restart. Let's delve into the detailed process and key aspects of how the WAL works in crash recovery.

**Code:**

```python
# Example Python-like pseudocode for logging a write operation to WAL
def log_to_wal(operation):
    wal_file.write(f"Op: {operation}\n")

def apply_log_entry(entry):
    # Code to apply the logged entry to database files
    pass

# Example sequence of operations during crash recovery
log_to_wal("Insert 10 into table A")
log_to_wal("Update B column in row X with value Y")

# Simulate a crash after writing logs but before applying them
crash()

# Recovery process:
for entry in wal_file.read():
    apply_log_entry(entry)
```

**Example input / output:**

| Input (Example)                          | Output (Expected)                  |
|------------------------------------------|-----------------------------------|
| Log to WAL: Insert 10 into table A       | Wal log entry recorded            |
| Log to WAL: Update B column in row X with value Y | Wal log entry recorded          |
| Crash after writing logs but before applying them | Rollback all unapplied logs and apply them during recovery   |

**Data:**

```json
{
  "wal_entry": {
    "timestamp": "2023-10-05T14:30:00",
    "operation": "Insert 10 into table A"
  },
  "log_file_contents": [
    {"timestamp": "2023-10-05T14:30:00", "operation": "Insert 10 into table A"},
    {"timestamp": "2023-10-05T14:35:00", "operation": "Update B column in row X with value Y"}
  ]
}
```

