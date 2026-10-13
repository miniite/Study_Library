---
type: "practice"
difficulty: "Deep"
---

# Explain how a bloom filter reduces read amplification for negative lookups in an LSM. If your false-positive rate is 1% and you have 6 SSTables to check, what's the expected number of wasted disk reads for a missing key?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Storage in LSM Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain how a bloom filter reduces read amplification in an LSM (Log-Structured Merge) database for negative lookups, we need to understand both the concept of bloom filters and how they integrate with LSMs. A bloom filter is a probabilistic data structure that can tell you if an element definitely does not exist or may have existed in a set. In the context of LSMs, it helps reduce unnecessary reads by allowing quick negative lookups.

In an LSM database, data is initially written to memory (a memtable) and periodically flushed to disk as SSTables. When a read operation occurs, the system first checks if the key might exist using a bloom filter before checking the SSTables. If the bloom filter indicates that the key definitely does not exist, no further reads are performed.

Let's break down the solution step by step:

Bloom Filter Basics: A bloom filter uses multiple hash functions to map keys into bit positions in an array. When inserting a key, each hash function sets the corresponding bit to 1. To check for membership (or non-membership), we apply the same hash functions and see if all bits are set to 1; if any is not set, the key definitely does not exist.

False-Positive Rate: The false-positive rate of a bloom filter depends on the number of elements inserted into it and the size of the bit array. With a 1% false-positive rate and 6 SSTables, we need to calculate how many keys will be incorrectly reported as possibly present but actually not.

Expected Number of Wasted Reads: For a missing key, if the bloom filter indicates that the key might exist, all 6 SSTables must still be checked. If it correctly reports that the key does not exist, no further reads are performed. We need to calculate the probability that a missing key is incorrectly reported as possibly present and multiply by the number of SSTables.

Let's assume we have ( n ) keys in total and each key has a 1% chance of being a false positive when checked against the bloom filter. The expected number of wasted disk reads can be calculated using the following steps:

False Positive Probability: For each missing key, there is a 1% probability that it will trigger a false positive.
Wasted Reads per False Positive: If a false positive occurs, all 6 SSTables will need to be checked.

Given these probabilities: [ \text{Expected number of wasted reads} = \frac{\text{Total keys}}{n} \times (1 - \text{False positive rate}) \times \text{Number of SSTables} ]

For a false-positive rate of 1% and 6 SSTables, the calculation is: [ \text{Expected number of wasted reads per missing key} = 0.01 \times 6 = 0.06 ]

Thus, for every missing key, there are expected to be ( 0.06 ) wasted disk reads.

In summary, using a bloom filter with a 1% false-positive rate and checking against 6 SSTables means that on average, we expect 0.06 additional disk reads per missing key due to false positives.

**Comparison:**

```
False Positive Rate	Number of SSTables	Expected Wasted Reads
0.01	6	0.06
```

**Code:**

```python
# Pseudocode for expected number of wasted reads calculation
def calculate_wasted_reads(false_positive_rate, num_sstables):
    # Calculate the expected number of wasted reads
    return false_positive_rate * num_sstables

# Given values
false_positive_rate = 0.01
num_sstables = 6

# Expected number of wasted reads per missing key
expected_wasted_reads = calculate_wasted_reads(false_positive_rate, num_sstables)
print(expected_wasted_reads)  # Output: 0.06
```

**Example input / output:**

```text
Input:
- false_positive_rate = 0.01
- num_sstables = 6

Output:
- Expected wasted reads per missing key: 0.06
```

**Data:**

```json
{
  "false_positive_rate": 0.01,
  "num_sstables": 6
}
```

**Flow:**

1. Yes
2. No
3. Missing Key Check
4. Bloom Filter Check
5. Key Present?
6. Check SSTables
7. No Further Reads
8. Wasted Reads (1 per SSTable)

