---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd size a bloom filter for an LSM: bits per key vs false-positive rate vs memory budget. If RAM is scarce and the workload is mostly existing-key reads, is investing in bloom filters worth it? Reason it out.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Storage in LSM Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To size a Bloom filter for an LSM (Log-Structured Merge Tree) database, we need to balance multiple factors: the number of bits per key, the false-positive rate, and the available memory budget. The core idea is to minimize the total storage footprint while ensuring acceptable performance and accuracy in read operations.

First, let's understand what a Bloom filter does: it's a probabilistic data structure that tests whether an element is a member of a set. It can produce false positives but never false negatives. In the context of an LSM database, we use bloom filters to quickly determine if a key exists before performing expensive disk I/O operations.

The formula for calculating the number of bits needed per key in a Bloom filter given n keys and a target false-positive rate (FP) is: [ m = -\frac{n \ln (FP)}{(\ln 2)^2} ] where ( m ) is the bit array size. This formula comes from the properties of the Bloom filter, which uses hash functions to map each key into multiple bits.

To reason through this step-by-step:

Understanding the Parameters:

n is the total number of keys.
FP is the desired false-positive rate. A lower FP means more accurate filtering but requires a larger bit array and more hash functions, leading to higher memory usage.
The goal is to keep m (bits per key) low while managing the FP.

Memory Budget Considerations:

If RAM is scarce, we need to carefully balance between the size of our Bloom filter and other critical data structures like merge trees or index structures.

#### Trade-offs
with Existing-Key Reads:

In an LSM database where most reads are for existing keys (i.e., hot data), a Bloom filter can significantly reduce disk I/O by quickly filtering out non-existent keys.
The cost of false positives is relatively low since they will result in additional, albeit minimal, I/O operations on the merge trees.

Calculating Bits per Key:

For example, if we have 1 million keys and want a false-positive rate of 0.01% (0.0001), we can calculate ( m ): [ m = -\frac{1,000,000 \ln(0.0001)}{(\ln 2)^2} \approx 13,849 \text{ bits} ]
This means each key would require about ( 13,849 / 1,000,000 = 0.013849 ) bits in the Bloom filter.

Practical Considerations:

We should also consider using multiple hash functions (usually 2-6) to reduce FP rate further at the cost of more memory per key.
Adjusting FP can help optimize for different use cases: lower FP means better filtering but higher memory, and vice versa.

To summarize, in a scenario with scarce RAM and mostly existing-key reads, investing in Bloom filters is generally worth it because they significantly reduce unnecessary disk I/O operations. However, the exact bit allocation should be carefully calculated based on the key count and acceptable false-positive rate to ensure efficient use of limited memory resources.

**Comparison:**

```
Parameters	Values
Keys Count	1M
FP Rate	0.01%
Bits per Key	~13,849
```

**Code:**

```python
def calculate_bits_per_key(n_keys, fp_rate):
    from math import log

    m = - (n_keys * log(fp_rate)) / (log(2) ** 2)
    return m

# Example calculation for 1M keys with 0.01% FP rate
bits_per_key = calculate_bits_per_key(1_000_000, 0.0001)
print(f"Bits per key: {bits_per_key:.4f}")
```

**Example input / output:**

```text
- **Input**:
  - `n_keys`: 1,000,000
  - `fp_rate`: 0.0001

- **Output**:
  - `bits_per_key`: 13849 bits (per key)
```

**Data:**

```json
{
  "keys_count": 1000000,
  "false_positive_rate": 0.0001,
  "bits_per_key": 13849
}
```

**Flow:**

1. Yes
2. No
3. Calculate bits_per_key
4. Is RAM scarce?
5. Consider Bloom filters for readoptimization
6. Other optimizations might bemore critical
7. Adjust FP rate to balanceaccuracy and memory usage
8. Implement with multiple hashfunctions if needed

