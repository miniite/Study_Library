---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd debug an LSM store where disk usage suddenly doubled overnight with no traffic change. Consider stuck compaction, snapshot pinning old SSTables, and WAL retention. Give the order you'd check them.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ LSM Write Path WAL and Compaction](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/LSM_Write_Path_WAL_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this problem, we need to understand that an LSM (Log-Structured Merge) store is designed for high write throughput but can suffer from disk space issues if not managed properly. The sudden doubling of disk usage without a corresponding increase in traffic suggests a problem with compaction or retention strategies. Let's break down the steps and tools we would use to debug this issue.

First, it’s important to understand that an LSM store manages data by writing all writes directly to memory (memtable) and periodically flushing these memtables to disk as SSTables. Over time, the number of SSTables can grow, leading to inefficiencies unless compaction is performed regularly. Compaction merges smaller SSTables into larger ones, reducing the overall number of files but increasing I/O during the process.

The question asks us to consider three potential issues: stuck compaction, snapshot pinning old SSTables, and write-ahead logging (WAL) retention. We'll check these in a specific order to ensure we address each issue methodically.

Stuck Compaction: This is where the compactor gets stuck and doesn’t complete its task, leading to an accumulation of small SSTables that are not being merged.
Snapshot Pinning Old SSTables: Snapshots can pin old SSTables in memory, preventing them from being deleted even if they should be eligible for removal during compaction.
WAL Retention: Write-ahead logs (WALs) also contribute to disk usage; they are temporary and usually flushed out once the transaction is committed, but they could accumulate if there's an issue with their cleanup.
Step-by-Step Debugging Order

Check Compactor Status:

Use tools like system_usage or equivalent monitoring metrics in your database system to check for compaction status.
Look at the log files for any indications of stuck compactions or errors during compaction attempts.

Verify Snapshots and Pinning:

Check if there are any snapshots that might be pinning SSTables older than expected. You can use commands like sstables to inspect snapshot metadata.
Verify the retention policy for snapshots to ensure old snapshots are being cleaned up properly.

Review WAL Retention:

Inspect the WALs and their cleanup policies to ensure they are not accumulating beyond necessary limits.
Check if there are any configuration changes or errors that might be causing excessive WAL creation.

#### Example
Commands and Tools
Compactor Status: system_usage in Cassandra
Snapshot Metadata: sstables --meta <table_name> in Cassandra
WAL Inspection: nodetool dumplogsummary in Cassandra to check the size of current and archived logs
Sample Debugging Workflow

Check Compaction Logs:

SH
nodetool compactionhistory

This command provides details on recent compactions, including any stuck or failed operations.

Verify Snapshot Retention:

SH
sstables --meta <keyspace_name:table_name>

Ensure that the retention policy is set correctly and old snapshots are being removed as expected.

Inspect WALs:

SH
nodetool dumplogsummary

This will give you an idea of how much space WALs are consuming and if they are being cleaned up properly.

#### Closing Summary

By following this structured approach, we can effectively diagnose why the disk usage has doubled in our LSM store. The order of checking is crucial to ensure we don't miss any critical issues that could be affecting performance or storage efficiency.

#### Common mistakes
include:

Overlooking compaction status and assuming it’s always running smoothly.
Ignoring snapshot management policies, leading to unnecessary pinning of SSTables.
Failing to monitor WAL retention, which can silently grow indefinitely if not configured correctly.

**Comparison:**

    | Compaction ID | Status   | Start Time  | End Time     |
    | ------------- | -------- | ----------- | ------------ |
    | 12345         | Stuck    | 09/15 18:00 | Not completed|

**Code:**

```
SH
Copy
# Sample commands for monitoring and debugging

# Compaction history
nodetool compactionhistory

# Snapshot metadata inspection
sstables --meta keyspace_name:table_name

# WALs summary
nodetool dumplogsummary
```

**Example input / output:**

```text
Input:
- Compaction history output shows no recent compactions.
- Snapshot metadata indicates old snapshots are still pinned.

Output:
- Investigate why compactor is stuck and resolve any issues.
- Verify snapshot pinning policy and ensure it's configured correctly to clean up old snapshots.
```

**Data:**

```yaml
# Example snapshot metadata output
keyspace_name:
  table_name:
    - sstable_id_1: {min_timestamp: 1609459200, max_timestamp: 1609873999, partition_count: 1000}
    - sstable_id_2: {min_timestamp: 1609874000, max_timestamp: 1610287999, partition_count: 1200}
```

**Flow:**

1. Check Compactor Status
2. Verify Snapshots
3. Inspect WALs
4. Resolve Issues

