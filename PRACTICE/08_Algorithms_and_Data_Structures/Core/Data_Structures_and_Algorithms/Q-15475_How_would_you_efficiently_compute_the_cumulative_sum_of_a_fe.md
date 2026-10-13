---
type: "practice"
difficulty: "Basic"
---

# How would you efficiently compute the cumulative sum of a feature array, and why is prefix sum an O(n) one-pass operation?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Complexity of Encodings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Complexity_of_Encodings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently compute the cumulative sum of a feature array, we use a technique called "prefix sum." The key idea here is that instead of computing the sum from scratch every time, we build up the cumulative sums incrementally in one pass through the data. This approach ensures an O(n) time complexity, making it very efficient.

The interviewer is probing your understanding of basic algorithmic techniques and their efficiency. You should assume a standard Python environment with lists or arrays as your primary data structures. Prefix sum can be applied to both integer and floating-point numbers.

Step-by-Step Solution
Initialization: Start by initializing an empty prefix sum array.
First Element Setup: Set the first element of the prefix sum array equal to the first element of the input array.
Iteration and Accumulation: Iterate through the remaining elements of the input array, setting each prefix sum value as the sum of the current element and the previous prefix sum value.

Let's go through these steps in detail:

Initialization:

Create an empty list prefix_sum with the same length as the input list feature_array.

First Element Setup:

Set prefix_sum[0] = feature_array[0].

Iteration and Accumulation:

For each element from index 1 to n-1, set prefix_sum[i] = prefix_sum[i-1] + feature_array[i].

This process ensures that you only need a single pass through the data structure to compute all cumulative sums.

Full Code Example
PYTHON
def compute_prefix_sum(feature_array):
    if not feature_array:
        return []

    # Initialize the prefix sum array with the same length as input array
    prefix_sum = [0] * len(feature_array)

    # Set the first element of the prefix sum array
    prefix_sum[0] = feature_array[0]

    # Compute the cumulative sums in one pass
    for i in range(1, len(feature_array)):
        prefix_sum[i] = prefix_sum[i-1] + feature_array[i]

    return prefix_sum

# Example usage:
feature_array = [1, 2, 3, 4, 5]
prefix_sum = compute_prefix_sum(feature_array)
print(prefix_sum)  # Output: [1, 3, 6, 10, 15]

#### Closing Summary
and Common Mistakes

By using the prefix sum technique, we ensure that the cumulative sums are computed in linear time O(n). The primary mistake to avoid is re-computing the sum from scratch each time, which would result in a quadratic time complexity. Always strive for efficient algorithms by leveraging incremental calculations.

Common Follow-Ups
Space Complexity: Can you further optimize this algorithm to use less space?
Edge Cases: What if the input array is empty? How do you handle such cases?

