---
type: "practice"
difficulty: "Deep"
---

# Explain the read path for a key in an LSM: memtable, immutable memtables, then L0..Ln with bloom + index. For a key present only in L3, count exactly how many components are consulted and where caches short-circuit.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Storage in LSM Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an LSM (Log-Structured Merge) database, reads typically involve multiple steps to find a specific key. The read path starts with checking the memtable(s), then proceeds to immutable memtables if present, and finally traverses through L0..Ln levels. Each step involves bloom filters and indexes for efficient searching.

To illustrate this in detail, let's consider how a read operation would proceed when the target key is only found in L3 of an LSM tiered storage setup with three levels (L0, L1, L2, L3).

#### Triage
What the Question is Asking

The question asks for a detailed breakdown of the read path for a specific key. This involves understanding how data is stored across different layers and how each layer contributes to finding the key efficiently. We need to count the number of components involved in the process and identify where caches might provide performance improvements.

#### First-Principles Solution
Check Memtable(s): The first step is to check any active memtables (memtable(s) that are currently being written into). If the key is found here, we're done.
Check Immutable Memtables: If not found in the memtable(s), and if there are immutable memtables present, these are also checked for the presence of the key. Immutable memtables are a historical snapshot of the memtable data.
Read L0-Ln Levels:
Bloom Filters: Each level (L0..Ln) starts with a bloom filter, which allows us to quickly determine if the key might exist in this level. If the bloom filter indicates that the key could be present, we proceed to the index for further refinement.
Index Lookup: Using the index from the bloom filter result, we locate the SSTables (Sorted String Tables) where the key is likely to reside. Each SSTable contains a segment of data sorted by keys.
Data Retrieval: Once the appropriate SSTable(s) are identified, their contents are loaded and searched linearly or using an index to find the exact location of the key.
Counting Components

For a key present only in L3:

Memtable Check (0 components): Not applicable here.
Immutable Memtables Check (n components, where n is the number of immutable memtables): Not applicable here as they are not present.
Bloom Filter Checks:
L0: 1 bloom filter
L1: 1 bloom filter
L2: 1 bloom filter
L3: 1 bloom filter (total of 4 bloom filters consulted)
Index Lookups:
L0: 1 index lookup
L1: 1 index lookup
L2: 1 index lookup
L3: 1 index lookup (total of 4 index lookups)
Caches and Short-Circuiting
Bloom Filters Cache: If the bloom filters are cached, they can significantly reduce the number of SSTable reads by quickly eliminating levels where the key does not exist.
Index Caches: Similarly, caching indices for each level can prevent redundant index lookups.

In summary, a key present only in L3 involves consulting 4 bloom filters and 4 index lookups. Caching mechanisms like bloom filter caches and index caches help optimize performance by reducing unnecessary read operations.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The read path for an LSM database is efficient due to the combination of bloom filters, indexes, and caching. Key components such as memtables, immutable memtables, L0-Ln levels, bloom filters, and indices all play crucial roles in ensuring fast reads. Common mistakes include overlooking the importance of bloom filter caches or not understanding how index lookups work across different tiers.

#### Follow-ups
could involve discussing how to tune these components for better performance, such as adjusting bloom filter false positive rates or optimizing cache eviction policies.

**Comparison:**

```
tableChart
row Level BloomFilter Result Index Lookup SSTable Data
row 0 true present sstable_data
row 1 false absent -
row 2 false absent -
row 3 true present sstable_data
```

**Code:**

```python
# Example Python code to simulate bloom filter and index lookup (simplified)
class BloomFilter:
    def __init__(self, size):
        self.size = size

    def add(self, key):
        pass  # Add logic here

    def test(self, key):
        return True  # Simulate a positive test result

# Simulating an index structure
class Index:
    def get_sstable(self, key):
        return "sstable_data"  # Simulated SSTable data retrieval

def read_path(key):
    bloom_filters = [BloomFilter(1024) for _ in range(4)]
    indices = [Index() for _ in range(4)]

    # Check memtables
    if not is_in_memtable(key): pass  # Skip as key not present here

    # Check immutable memtables (not applicable here)
    if not are_in_immutable_memtables(key): pass  # Skip as they are not present

    # Bloom filter checks for L0-L3 levels
    bloom_filter_results = [bf.test(key) for bf in bloom_filters]

    # Index lookups based on bloom filter results
    for i, result in enumerate(bloom_filter_results):
        if result:
            sstable_data = indices[i].get_sstable(key)
            process_data(sstable_data)

# Function to simulate key presence in memtables (not present here)
def is_in_memtable(key):
    return False

# Function to simulate key presence in immutable memtables (not applicable here)
def are_in_immutable_memtables(key):
    return False
```

**Example input / output:**

| Level | Bloom Filter Test Result | Index Lookup | SSTable Data |
|-------|-------------------------|--------------|--------------|
| L0    | True                    | Present      | sstable_data |
| L1    | False                   | Absent       | -            |
| L2    | False                   | Absent       | -            |
| L3    | True                    | Present      | sstable_data |

**Data:**

```json
{
  "key": "user_12345",
  "bloom_filters": [
    { "size": 1024 },
    { "size": 1024 },
    { "size": 1024 },
    { "size": 1024 }
  ],
  "indices": [
    { "level": 0 },
    { "level": 1 },
    { "level": 2 },
    { "level": 3 }
  ]
}
```

**Flow:**

1. No
2. No
3. True
4. False
5. True
6. False
7. True
8. False
9. True
10. Start
11. Is key in memtable?
12. Are there immutablememtables?
13. Check L0 bloom filter
14. Check L0 index, fetch SSTabledata
15. Process data
16. Check L1 bloom filter
17. Check L1 index, fetch SSTabledata
18. Process data
19. Check L2 bloom filter
20. Check L2 index, fetch SSTabledata
21. Process data
22. Check L3 bloom filter
23. Check L3 index, fetch SSTabledata
24. Process data

