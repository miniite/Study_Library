---
type: "practice"
difficulty: "Intermediate"
---

# A batch dedup of 800M training records using a Python set OOMs on the training box; how would you re-architect it with external sorting or a Bloom filter, and what false-positive rate is acceptable for training data?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ External Memory Deduplication](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/External_Memory_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of out-of-memory (OOM) errors when performing batch deduplication on 800 million training records using Python sets, we need to re-architect our approach. The key challenge is that a set in Python requires storing all elements in memory, which can quickly consume significant resources as the dataset grows. We'll explore two strategies: external sorting and Bloom filters.

The core of this problem revolves around finding a way to deduplicate records without keeping all data in memory at once. External sorting allows us to process large datasets by breaking them into smaller chunks that fit into memory, while Bloom filters offer a probabilistic approach with a trade-off between accuracy and memory usage.

**Comparison:**

```
Data Structure	Memory Usage	False-Positive Rate	Example Scenario
Python Set	High	0	Not feasible for large datasets due to memory constraints.
External Sort	Moderate	0	Effective for sorting and deduplication in chunks.
Bloom Filter	Low	> 0 (controlled)	Efficient for large datasets with acceptable false positives.
```

**Code:**

```
Example Python code for external sorting

def read_large_file(file_path): chunk_size = 100000 # Adjust based on available memory records = [] with open(file_path, 'r') as file: while True: chunk = file.read(chunk_size) if not chunk: break for record in chunk.splitlines(): records.append(record) return records

def external_sort(records): # Simple bubble sort to demonstrate the concept n = len(records) for i in range(n): for j in range(0, n - i - 1): if records[j] > records[j + 1]: records[j], records[j + 1] = records[j + 1], records[j] return records

def deduplicate_with_external_sort(file_path): sorted_records = external_sort(read_large_file(file_path)) unique_records = [sorted_records[i] for i in range(len(sorted_records)) if i == 0 or sorted_records[i] != sorted_records[i - 1]] return unique_records

CODE
Copy

```python
# Example Python code for Bloom filter

from bitarray import bitarray

def create_bloom_filter(size, hash_count):
    bloom_filter = bitarray(size)
    bloom_filter.setall(0)
    return bloom_filter

def add_to_bloom_filter(bloom_filter, record_hash):
    for i in range(hash_count):
        index = hash(record_hash) % (len(bloom_filter))
        bloom_filter[index] = 1

def check_in_bloom_filter(bloom_filter, record_hash):
    for i in range(hash_count):
        index = hash(record_hash) % (len(bloom_filter))
        if not bloom_filter[index]:
            return False
    return True

# Example usage with a hypothetical dataset
hash_count = 5
bloom_filter_size = 1000000
bloom_filter = create_bloom_filter(bloom_filter_size, hash_count)

with open('training_data.txt', 'r') as file:
    for line in file:
        if not check_in_bloom_filter(bloom_filter, line):
            print(line)  # This would be the unique record after processing
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `read_large_file('training_data.txt')` | List of records, each representing a line in the file. |
| `external_sort(records)` | Sorted list of unique records based on the external sort algorithm. |
| `deduplicate_with_external_sort('training_data.txt')` | Unique sorted list of records. |

**Data:**

```json
{
  "hash_count": 5,
  "bloom_filter_size": 1000000,
  "training_data_file": "training_data.txt"
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Yes
6. No
7. Bloom Filter Setup
8. Add Record Hash
9. Mark as Seen
10. Check Record Hash
11. Is in Bloom?
12. Record is Duplicate, Discard
13. Record is New, Keep and Addto Bloom

