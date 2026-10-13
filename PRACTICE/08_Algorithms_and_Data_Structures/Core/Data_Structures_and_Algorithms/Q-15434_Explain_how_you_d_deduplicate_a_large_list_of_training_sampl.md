---
type: "practice"
difficulty: "Basic"
---

# Explain how you'd deduplicate a large list of training samples, and what data structure gives you an average O(1) membership check.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ External Memory Deduplication](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/External_Memory_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deduplicate a large list of training samples efficiently, we need to ensure that each sample is checked against previously seen ones without reprocessing all of them from scratch. The goal here is to achieve an average O(1) membership check, which can be accomplished using a hash set data structure.

#### Triaging the Question

The question asks for a method to remove duplicates from a list while ensuring fast membership checks (average O(1)). This implies that we need to choose a suitable data structure and understand how it operates under different scenarios. We'll focus on implementing this in Python, as familiarity with common programming languages is often assumed.

Step-by-Step Solution
Choose the Data Structure: A hash set (set in Python) is ideal for our needs because membership checks are typically O(1).
Algorithm:
Traverse through each sample.
Check if the sample exists in the set.
If it does not, add it to the set and include it in the result list.
Time Complexity: With a hash set, the average time complexity for both insertion and lookup is O(1). This makes the overall algorithm efficient.

#### Implementation
PYTHON
def deduplicate(samples):
    seen = set()
    unique_samples = []
    for sample in samples:
        if sample not in seen:
            seen.add(sample)
            unique_samples.append(sample)
    return unique_samples

# Example usage
samples = [1, 2, 3, 4, 5, 3, 6, 7, 8, 9, 10, 10]
unique_samples = deduplicate(samples)
print(unique_samples)  # Output: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

#### Closing Summary
and Common Mistakes

This approach ensures that each sample is checked against the set in constant time on average. The main challenge lies in handling edge cases such as empty input lists or very large datasets where memory usage could be a concern.

#### Common Mistakes

Failing to use a hash set, leading to slower checks.
Not considering the space complexity implications of storing all previous samples.
Misunderstanding the O(1) average time complexity and mistaking it for worst-case constant time.

#### Conclusion

Using a hash set provides an efficient solution with O(1) average membership checks. The choice of this data structure is key to achieving the desired performance, especially when dealing with large datasets.

**Code:**

```python
def deduplicate(samples):
    seen = set()
    unique_samples = []
    for sample in samples:
        if sample not in seen:
            seen.add(sample)
            unique_samples.append(sample)
    return unique_samples
```

**Example input / output:**

```text
samples = [1, 2, 3, 4, 5, 3, 6, 7, 8, 9, 10, 10]
unique_samples = deduplicate(samples)
print(unique_samples)  # Output: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

