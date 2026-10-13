---
type: "practice"
difficulty: "Basic"
---

# How would you store and look up embeddings for millions of items, and why is an array indexed by integer ID efficient?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Arrays and Embedding Lookup](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays_and_Embedding_Lookup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently store and look up embeddings for millions of items, we typically use a data structure that allows for fast retrieval based on unique identifiers (IDs). For ML engineers, one of the most straightforward and efficient methods is to use an array indexed by integer ID. This approach leverages the speed of direct memory access provided by arrays.

#### Triage

The question essentially asks how we can store embeddings in a way that allows quick lookups using their item IDs. The interviewer is probing your understanding of data structures, specifically arrays and hash tables, and your ability to reason about time complexity.

#### First-Principles Solution

Given the requirement to store millions of items efficiently, let's break down why an array indexed by integer ID can be a good choice:

Direct Memory Access: Arrays provide constant-time access (O(1)) via direct indexing. This means that if we have the item’s ID, we can directly compute its memory location.
Simplicity and Speed: Directly accessing an array element is faster than other data structures like hash tables or trees, which typically offer average (O(1)) but could degrade to (O(n)) in the worst case.
Detailed Steps

Step 1: Define Data Structure
Each item will have a unique integer ID and its corresponding embedding (vector of numbers). The array will be large enough to hold all possible IDs, with each index containing an embedding.

Step 2: Memory Allocation
Allocate memory for the entire array at once. This can be done dynamically based on expected growth.

Step 3: Lookups and Inserts
For lookups or inserts, we use the item ID directly as the index into the array to access or modify the embedding.

#### Example
Code
PYTHON
# Python example for storing and looking up embeddings in an array

class EmbeddingsArray:
    def __init__(self, num_items):
        self.embeddings = [None] * num_items  # Initialize with None

    def add_embedding(self, item_id, embedding):
        if not (0 <= item_id < len(self.embeddings)):
            raise IndexError("Item ID out of bounds")
        self.embeddings[item_id] = embedding

    def get_embedding(self, item_id):
        if not (0 <= item_id < len(self.embeddings)):
            raise IndexError("Item ID out of bounds")
        return self.embeddings[item_id]

#### Closing Summary

Using an array indexed by integer IDs is efficient for storing and looking up embeddings because it provides constant-time access. This method ensures quick lookups, making it suitable for applications where performance is critical.

#### Common Mistakes/Follow-Ups
Mistake 1: Not considering memory overhead. While arrays are fast, they might consume more memory compared to other data structures.
Mistake 2: Failing to validate the item ID range before accessing or modifying the array.

If asked about alternatives, you could discuss hash tables which offer average (O(1)) time complexity but may have collisions and require additional management overhead. Trees like binary search trees can also provide (O(\log n)) access times, but they are generally slower than arrays for large datasets.

**Code:**

```python
# Python example code
```

