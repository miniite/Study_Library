---
type: "practice"
difficulty: "Expert"
---

# Explain how Delta Lake and Iceberg differ in their concurrency-control mechanism for concurrent writers and how each handles a compaction running at the same time as an append without corrupting a reader's snapshot.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Optimistic Concurrency and Clocks](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Delta Lake and Iceberg are both distributed storage systems designed to support ACID transactions in data lakes, but they handle concurrency control differently, especially during compaction operations. In this answer, we'll explore how these two frameworks manage concurrent writers and compactions without corrupting a reader's snapshot.

#### Triage
Understanding the Core Problem

The question is probing your understanding of concurrency control mechanisms in distributed storage systems, specifically focusing on Delta Lake and Iceberg. We need to understand their underlying data models and how they handle concurrent writes and compaction operations. The key assumptions here are that both systems support ACID transactions and manage snapshot isolation.

#### First-Principles Solution
Delta Lake

Delta Lake uses a log-based approach for concurrency control, maintaining an immutable transaction log (change log) to ensure strong consistency across writers and readers. During an append operation, Delta Lake first logs the changes in the transaction log before applying them to the data files. This ensures that no reader sees incomplete writes. Compactions in Delta Lake can run concurrently with appends because compaction operations only modify existing data files and do not introduce new write transactions. Thus, a reader will always see either an unmodified snapshot or one fully updated by all concurrent writes.

Iceberg

Iceberg uses a hybrid approach combining transaction logs and metadata versioning to handle concurrency. When a writer starts a transaction, it locks the relevant segments to prevent other writers from modifying them. During compaction, Iceberg also needs to ensure that no reader sees incomplete data changes. Iceberg achieves this by maintaining metadata versions and ensuring that only consistent snapshots are exposed to readers.

Detailed Explanation
Delta Lake's Concurrency Control Mechanism
Log-Based Appends: When a writer starts an append operation, it first logs the new file paths in the transaction log.
Apply Changes Synchronously: The writer then synchronously writes the data files and updates the manifest files to include these new paths.
Compaction Operations: Compaction operations run on existing data files without introducing new write transactions. They only rewrite or merge existing segments, ensuring that readers always see a consistent snapshot.
Iceberg's Concurrency Control Mechanism
Transaction Locking: When starting a transaction, the writer locks specific segments to prevent other writers from modifying them.
Metadata Versioning: Iceberg maintains multiple metadata versions and ensures that only fully committed transactions are exposed to readers.
Compaction Operations: During compactions, Iceberg needs to ensure that ongoing reads do not see partial data changes. This is achieved by managing metadata consistency and ensuring that any read sees either a consistent snapshot or the result of all completed transactions.

#### Closing Summary

Delta Lake and Iceberg both handle concurrency in distributed storage systems effectively but use different mechanisms. Delta Lake relies on a log-based approach to ensure strong consistency, making compactions safe for concurrent appends. In contrast, Iceberg uses transaction locking and metadata versioning to manage concurrent writers and compactions, ensuring that readers always see valid snapshots.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that Delta Lake's log-based approach will lead to inconsistent data during compaction.

#### Follow-Up
Discuss the trade-offs between log-based systems like Delta Lake and transactional locking in Iceberg. Explore scenarios where these differences might impact performance or data integrity.

**Comparison:**

```
Operation	Delta Lake	Iceberg
Append	Log change, write data, update manifest	Lock segments, compact files, update metadata version
Compaction	No new writes, rewrite existing segments	Locks segments, runs compactions, updates metadata consistently
```

**Code:**

```python
# Example of a simple Delta Lake append operation
def delta_append(data):
    with open_transaction_log():
        log_change(data.file_path)
    write_data_file(data)
    update_manifest(data.file_path)

# Example of an Iceberg compaction operation
def iceberg_compact(segment):
    lock_segment(segment)  # Lock segment to prevent writes
    compact_segment_files(segment)  # Perform compaction on locked segments
```

**Example input / output:**

```text
Input (Delta Lake):
- Append a new file with path "path/to/new/file1"
- Compaction running on existing segments

Output:
- Transaction log updated with new file path
- New data files written to storage
- Manifest updated to include new paths

Input (Iceberg):
- Writer starts transaction, locks segment
- Compaction operation runs on locked segment
- Metadata version updated post-compaction

Output:
- Segment metadata ensures consistent snapshot is visible
- Readers see fully committed transactions or the current snapshot
```

**Data:**

```json
{
  "deltaLakeTransaction": {
    "operation": "append",
    "file_paths": ["path/to/new/file1", "path/to/new/file2"]
  },
  "icebergCompaction": {
    "segment": "segment_id_123",
    "locked": true,
    "compacted_files": ["new_path_1", "new_path_2"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start Transaction
4. Lock Segment?
5. Run Compaction
6. Append New Data Files
7. Update Metadata Version
8. Log Changes in Manifest
9. Compacted Files Written toStorage

