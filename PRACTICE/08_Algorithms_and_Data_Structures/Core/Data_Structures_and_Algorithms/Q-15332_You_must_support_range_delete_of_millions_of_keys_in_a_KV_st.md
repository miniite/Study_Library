---
type: "practice"
difficulty: "Deep"
---

# You must support 'range delete' of millions of keys in a KV store without a long stop-the-world scan; how does a tombstone-and-compaction design differ algorithmically from deleting one node at a time?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of efficiently supporting 'range delete' operations in a key-value (KV) store without a long stop-the-world scan, we need to explore a design that uses tombstones and compaction. This approach differs significantly from deleting nodes one by one because it optimizes for performance by reducing write amplification and enabling efficient data recovery.

The core idea is that instead of immediately removing all keys in the range during a deletion operation, we mark them as "tombstone" entries. These tombstone markers serve as placeholders indicating the deletion intent without actually removing the data from disk. Later, during compaction, these tombstones are purged, and the associated space is reclaimed. This method allows us to perform deletions in constant time while ensuring that the system remains operational.

**Code:**

```python
class TombstoneEntry:
    def __init__(self, key):
        self.key = key
        self.deleted = False

# Example of a tombstone entry
tombstone_entry = TombstoneEntry('key1')
print(tombstone_entry.key)  # Output: 'key1'
```

**Example input / output:**

```
Input: delete_range('prefix1', 'prefix2')
Output: Marks all keys within the range as tombstones.
Example:
PLAINTEXT
Copy
Before:
KV Store:
key1 -> value1
prefix1key1 -> value2
prefix2key1 -> value3

After delete_range('prefix1', 'prefix2'):
KV Store with Tombstones:
key1 -> value1 (tombstone)
prefix1key1 -> value2 (tombstone)
prefix2key1 -> value3 (tombstone)
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is deletion range available?
7. Mark keys as tombstones
8. Skip, continue normaloperations
9. Compaction triggered
10. Are tombstones old enough tobe deleted?
11. Purge tombstones and reclaimspace
12. Wait for next compaction cycle

