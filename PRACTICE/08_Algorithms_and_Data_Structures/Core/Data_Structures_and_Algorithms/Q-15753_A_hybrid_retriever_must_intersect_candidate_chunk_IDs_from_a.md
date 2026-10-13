---
type: "practice"
difficulty: "Deep"
---

# A hybrid retriever must intersect candidate chunk IDs from a metadata filter with those from vector search before ranking; compare sorted-merge, hash-set probing, and roaring bitmaps for a filter that matches millions of chunks.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Diff and LCS Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Diff_and_LCS_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently intersect candidate chunk IDs from a metadata filter with those from vector search in a hybrid retriever, we need to consider the performance characteristics of different data structures and algorithms. Specifically, we will compare three approaches: sorted-merge, hash-set probing, and roaring bitmaps. Each approach has its strengths and trade-offs depending on the scale and requirements.

The question is essentially asking us to evaluate these techniques for intersecting two sets of chunk IDs (one from metadata filtering and one from vector search) when dealing with millions of chunks. The interviewer is likely interested in understanding how each method works, their time complexity, memory usage, and practical considerations for large-scale applications.

#### Triage
Question Core: Compare sorted-merge, hash-set probing, and roaring bitmaps for intersecting two sets of chunk IDs.

#### Assumptions
We have two sets of chunk IDs: one from metadata filtering (Set A) and one from vector search (Set B).
The size of each set can be in the millions.
Key Considerations: Time complexity, space complexity, memory usage, ease of implementation, and scalability.

#### First-Principles Solution

Sorted-Merge:

Algorithm: Merge two sorted arrays into a single sorted array by comparing elements from both sets one by one.
Time Complexity: (O(n + m)), where (n) and (m) are the sizes of Set A and Set B, respectively.
Space Complexity: (O(n + m)) for storing the merged result.
Advantages: Simple to implement. Efficient if both sets are already sorted.
Disadvantages: Requires additional space proportional to the size of the input.

Hash-Set Probing:

Algorithm: Use hash tables (hash sets) to store elements from one set and then check for membership in the other set.
Time Complexity: (O(n + m)) on average, with a constant factor determined by the load factor of the hash table.
Space Complexity: (O(max(n, m))) due to potential collisions and resizing overhead.
Advantages: Very fast lookups. Can be implemented using modern hash tables.
Disadvantages: May suffer from high memory usage if there are many collisions.

Roaring Bitmaps:

Algorithm: Use a specialized data structure optimized for representing sets of integers, particularly useful in scenarios involving large sparse ranges.
Time Complexity: Intersection operations can be performed efficiently using bitwise operations.
Space Complexity: Typically requires less memory than hash tables or sorted arrays, especially when dealing with large sparse ranges.
Advantages: Efficient intersection operations. Optimized for range queries and large datasets.
Disadvantages: Slightly more complex to implement compared to simple hash sets.

#### Closing Summary

In summary, the choice between these methods depends on the specific requirements of the application:

Sorted-Merge is straightforward but requires additional space proportional to the input size.
Hash-Set Probing offers fast lookups and can handle collisions effectively but may use more memory than necessary for large sets.
Roaring Bitmaps provide efficient intersection operations, especially suitable for scenarios with large sparse ranges.

#### Common mistakes
include underestimating the memory requirements of hash tables or overestimating the benefits of sorted arrays without considering additional space. Always consider the practical implications and trade-offs based on the specific use case.

#### Common Mistakes/Follow-Ups
Failing to recognize the importance of memory usage in large-scale applications.
Ignoring the overhead of implementing and maintaining hash sets, especially when dealing with high collision rates.
Overlooking the benefits of specialized data structures like roaring bitmaps for handling large sparse ranges efficiently.

**Code:**

```python
# Example implementation using Python

def sorted_merge(set_a, set_b):
    result = []
    i, j = 0, 0
    while i < len(set_a) and j < len(set_b):
        if set_a[i] == set_b[j]:
            result.append(set_a[i])
            i += 1
            j += 1
        elif set_a[i] < set_b[j]:
            i += 1
        else:
            j += 1
    return result

def hash_set_intersection(set_a, set_b):
    set_a_hash = set(set_a)
    intersection = [item for item in set_b if item in set_a_hash]
    return intersection

def roaring_bitmap_intersection(set_a, set_b):
    from roaringbitmap import RoaringBitmap

    set_a_bitmap = RoaringBitmap(set_a)
    intersection = set_a_bitmap.intersection(RoaringBitmap(set_b))
    return list(intersection)
```

**Example input / output:**

| Input | Output (Sorted-Merge) |
|-------|----------------------|
| Set A: [1, 2, 3, 4, 5] <br> Set B: [4, 5, 6, 7, 8] | [4, 5] |
| Input | Output (Hash-Set Intersection) |
|-------|--------------------------------|
| Set A: [1, 2, 3, 4, 5] <br> Set B: [4, 5, 6, 7, 8] | [4, 5] |
| Input | Output (Roaring Bitmap Intersection) |
|-------|--------------------------------------|
| Set A: [1, 2, 3, 4, 5] <br> Set B: [4, 5, 6, 7, 8] | [4, 5] |

**Data:**

```json
{
  "setA": [1, 2, 3, 4, 5],
  "setB": [4, 5, 6, 7, 8]
}
```

**Flow:**

1. Sorted-Merge
2. Yes
3. No
4. Yes
5. No
6. Set A
7. Are elements equal?
8. Add element to result
9. Increment index i or j?
10. Continue merging

