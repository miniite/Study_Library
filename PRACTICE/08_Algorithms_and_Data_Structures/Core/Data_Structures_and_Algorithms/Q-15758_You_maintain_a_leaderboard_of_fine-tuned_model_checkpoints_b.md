---
type: "practice"
difficulty: "Intermediate"
---

# You maintain a leaderboard of fine-tuned model checkpoints by eval score with frequent inserts and top-k reads; which structure (skip list, balanced tree, heap) fits and why does a re-sorted array fall over under load?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Heaps and Running Median](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heaps_and_Running_Median.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To maintain a leaderboard of fine-tuned model checkpoints by evaluation score with frequent inserts and top-k reads, we need to choose a data structure that supports both efficient insertions and retrievals. The options provided are skip list, balanced tree, heap, and the re-sorted array. We'll evaluate each one based on their strengths and weaknesses.

Firstly, let's understand what each structure offers:

Skip List: A probabilistic data structure that provides fast search within an ordered sequence of elements.
Balanced Tree (e.g., AVL or Red-Black Tree): A self-balancing binary search tree that maintains a logarithmic height, ensuring efficient operations.
Heap: Typically used for priority queues, where the root element always has the highest priority. Heaps support efficient insertions and deletions of the top element but are not as good for general order maintenance.
Re-sorted Array: A simple array that needs to be completely re-sorted every time an insertion occurs, which is inefficient.

For frequent inserts and reads of the top-k elements, a skip list or balanced tree would be more suitable. However, we'll delve deeper into their performance characteristics.

#### Triaging the Question

The core of this question lies in understanding:

The nature of insertions and reads.
The efficiency requirements for both operations.
Why re-sorted arrays are not ideal under load.

#### Assumptions
include frequent updates to the leaderboard (model checkpoints) with a need to efficiently retrieve top-k elements at any point in time.

#### First Principles Solution

Let's evaluate each structure:

Skip List:
Insertion: (O(\log n)) on average.
Read Top-k: (O(k + \log n)).
Balanced Tree (e.g., AVL, Red-Black Tree):
Insertion: (O(\log n)).
Read Top-k: (O(k + \log n)).

Both skip lists and balanced trees offer good performance for both insertions and top-k reads.

Heap:
Insertion: (O(\log n)).
Read Top-1 (or k): (O(1)) if using a max-heap, but maintaining the entire heap's order is complex.
Re-sorted Array:
Insertion: (O(n \log n)) to re-sort the array after each insertion.
Read Top-k: (O(k + \log n)).

#### Closing Summary
and Common Mistakes

The skip list or balanced tree is the best choice for maintaining a leaderboard with frequent inserts and top-k reads. Both offer efficient insertions and retrievals, making them suitable for real-time applications where updates are common.

#### Common Mistakes/Follow-Ups

Misunderstanding that re-sorted arrays require (O(n \log n)) time for each insertion.
Overlooking the importance of maintaining a balanced tree's properties (like height) in ensuring efficient operations.
Not considering the average-case and worst-case performance differences between skip lists and balanced trees.

In conclusion, skip list or balanced tree are ideal structures for this problem due to their efficient insertions and retrievals. Re-sorted arrays fall short under load due to their high insertion cost.

**Comparison:**

```
Structure	Insertion	Read Top-k
Skip List	O(log n)	O(k + log n)
Balanced Tree	O(log n)	O(k + log n)
Heap	O(log n)	O(1) (if max-heap)
Re-sorted Array	O(n log n)	O(k + log n)
```

**Code:**

```python
# Example Skip List Implementation (simplified)
class Node:
    def __init__(self, value):
        self.value = value
        self.forward = [None] * 16  # Skip list level

def insert(head, value):
    nodes_to_update = []
    n = head
    for i in range(len(n.forward) - 1, -1, -1):  # Traverse levels
        while n.forward[i] and n.forward[i].value < value:
            n = n.forward[i]
        nodes_to_update.append((i, n))

    insert_level = get_random_level()  # Random level for probabilistic nature

    new_node = Node(value)
    for level in range(min(insert_level, len(n.forward) - 1), -1, -1):
        new_node.forward[level] = nodes_to_update[level][0].forward[level]
        nodes_to_update[level][0].forward[level] = new_node
```

**Example input / output:**

```text
# Example input and output for top-1 read operation using a skip list
def find_top_k(node, k):
    result = []
    while node:
        if len(result) < k:
            result.append(node.value)
        node = node.forward[0]
    return sorted(result, reverse=True)[:k]

top_1 = find_top_k(head, 1)
print(top_1)  # Output: [95.2]
```

**Data:**

```json
{
    "model_checkpoints": [
        {"id": 1, "score": 95.2},
        {"id": 2, "score": 94.8},
        {"id": 3, "score": 94.5}
    ]
}
```

**Flow:**

1. Skip List
2. Insertion O(log n)
3. Read Top-k O(k + log n)
4. Balanced Tree
5. Insertion O(log n)
6. Read Top-k O(k + log n)
7. Heap
8. Insertion O(log n)
9. Read Top-1 O(1) (max-heap)
10. Re-sorted Array
11. Insertion O(n log n)
12. Read Top-k O(k + log n)

