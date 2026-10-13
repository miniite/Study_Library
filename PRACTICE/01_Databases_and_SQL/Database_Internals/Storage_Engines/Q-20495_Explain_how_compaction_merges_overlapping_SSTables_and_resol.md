---
type: "practice"
difficulty: "Deep"
---

# Explain how compaction merges overlapping SSTables and resolves duplicate keys by sequence number. Walk through what happens to a key that was written, deleted, then written again as it flows through compaction levels.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ LSM Write Path WAL and Compaction](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/LSM_Write_Path_WAL_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Compaction in databases like Cassandra or RocksDB is crucial for maintaining performance and data integrity, especially when dealing with Log-Structured Merge Trees (LSM). At a high level, compaction merges multiple SSTables into fewer ones to reduce the number of files on disk, improving read efficiency. When duplicate keys are involved—specifically, keys that have been written, deleted, then re-written—the sequence numbers play a critical role in resolving these conflicts.

#### Triaging the Question

The core question revolves around understanding how compaction handles overlapping SSTables and resolves duplicates using sequence numbers. The interviewer is probing your knowledge of LSM tree internals, particularly in handling write amplification issues like key deletions followed by new writes. We will assume that we are dealing with a typical scenario where data is written incrementally to multiple levels of the LSM tree.

#### First-Principles Solution
Understanding SSTables and Overlaps: In an LSM, data is initially stored in MemTables before being periodically flushed to SSTables (Sorted String Tables). Each SSTable represents a set of key-value pairs. When compaction runs, it merges multiple SSTables into fewer ones.
Sequence Numbers and Write Amplification: Sequence numbers are assigned to each write operation, ensuring that writes can be uniquely identified even when they overwrite previous versions of the same key.
Compaction Process:
Compaction identifies overlapping keys across multiple SSTables (e.g., keys with sequence numbers in different ranges).
It merges these keys into a single output SSTable while respecting their sequence numbers to resolve any conflicts.
Detailed Walkthrough

Let's consider a scenario where the key "user1" is written, deleted, and then re-written again.

Initial Writes:

At time ( t_1 ), user1: valueA is written.
At time ( t_2 ), user1: valueB overwrites valueA.
At time ( t_3 ), user1: valueC overwrites valueB.

Compaction Levels:

Assume these writes have been flushed to different SSTables.
SSTable A contains {user1: valueA, sequence: seq1}.
SSTable B contains {user1: valueB, sequence: seq2}.
SSTable C contains {user1: valueC, sequence: seq3}.

Compaction Process:

Compaction starts by reading the keys in all overlapping ranges from SSTables A, B, and C.
It then merges these keys into a new output SSTable D.
During merging, it uses the sequence numbers to resolve any conflicts. For example, if user1: valueB (seq2) overlaps with user1: valueC (seq3), compaction will retain only one version based on its sequence number.
Detailed Example

#### Cons
ider a simplified input scenario:

Input SSTables:
A: {user1: valueA, seq1}
B: {user1: valueB, seq2}
C: {user1: valueC, seq3}

Compaction process:

PLAINTEXT
Output SSTable D:
- user1: valueA (seq1) -> retained because seq1 < seq2 and seq3
- user1: valueC (seq3) -> retained because seq3 > seq2 and seq1

#### Closing Summary
and Common Mistakes/Follow-Ups

Compaction ensures that data is merged correctly by leveraging sequence numbers to resolve conflicts. The process involves reading overlapping keys from multiple SSTables, merging them into a single output SSTable, and retaining the latest version based on sequence numbers.

#### Common Mistakes

Failing to account for write amplification issues like key deletions.
Misunderstanding how sequence numbers are used during compaction to resolve conflicts.
Overlooking the importance of maintaining the correct order of writes and their sequence numbers.

#### Follow-Ups

How does this process handle concurrent writes?
What happens if multiple keys with the same sequence number are written?
Can you discuss the performance implications of frequent compactions?

**Comparison:**

| Key | Value | Sequence |
| --- | ----- | -------- |
| user1 | valueA | 1        |
| user2 | valueX | 5        |
| user1 | valueB | 3        |
| user3 | valueY | 7        |
| user1 | valueC | 4        |

**Code:**

```python
# Example pseudo-code for compaction logic
def compact_sstables(sstables):
    output_table = {}
    for sstable in sstables:
        for key, value, seq in sstable.entries():
            if key not in output_table or seq > output_table[key]['seq']:
                output_table[key] = {'value': value, 'seq': seq}
    return list(output_table.items())

# Sample SSTable entries
sstables = [
    [('user1', 'valueA', 1), ('user2', 'valueX', 5)],
    [('user1', 'valueB', 3), ('user3', 'valueY', 7)],
    [('user1', 'valueC', 4)]
]

output = compact_sstables(sstables)
print(output)  # Expected output: [('user1', 'valueC'), ('user2', 'valueX'), ('user3', 'valueY')]
```

**Example input / output:**

```text
Input SSTables:
- A: [ ('user1', 'valueA', 1), ('user2', 'valueX', 5) ]
- B: [ ('user1', 'valueB', 3), ('user3', 'valueY', 7) ]
- C: [ ('user1', 'valueC', 4) ]

Output SSTable D:
[ ('user1', 'valueC'), ('user2', 'valueX'), ('user3', 'valueY') ]
```

**Data:**

```json
{
    "sstables": [
        {
            "key": "user1",
            "value": "valueA",
            "seq": 1
        },
        {
            "key": "user2",
            "value": "valueX",
            "seq": 5
        }
    ],
    "sstables": [
        {
            "key": "user1",
            "value": "valueB",
            "seq": 3
        },
        {
            "key": "user3",
            "value": "valueY",
            "seq": 7
        }
    ],
    "sstables": [
        {
            "key": "user1",
            "value": "valueC",
            "seq": 4
        }
    ]
}
```

**Flow:**

1. Read SSTable A
2. Merge with SSTables B and C
3. Merge keys based onsequence numbers
4. Output SSTable D
5. Retain latest version of eachkey

