---
type: "practice"
difficulty: "Deep"
---

# How would you keep a running list of the k highest-scoring retrieved passages while scanning candidates once, and why does a min-heap of size k work?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Heaps and Running Median](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heaps_and_Running_Median.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To maintain a running list of the (k) highest-scoring retrieved passages while scanning through all candidate passages only once, using a data structure like a min-heap is an efficient approach. The core idea is to ensure that at any point in time, we have the (k) best passages from the ones seen so far. This problem can be effectively addressed by leveraging the properties of a min-heap.

#### Triage

The question asks how to keep track of the highest-scoring (k) passages with a single scan through all candidates. The key is to use a min-heap (priority queue) of size (k). Specifically, we need to:

Understand what a heap does.
Explain why using a min-heap is beneficial.
Describe the algorithm step-by-step.

#### Assumptions
and First Principles
We are scanning through a list of passages in one pass.
Each passage has a score associated with it.
The goal is to maintain the (k) highest-scoring passages seen so far.
A min-heap is used because its properties allow for efficient management of the (k) largest elements.

#### Solution
Initialization: Start by initializing an empty min-heap.
First (k) Passages: As we encounter the first (k) passages, insert them into the heap. This step ensures that our heap contains the initial highest-scoring candidates.
Subsequent Passages: For each additional passage, compare its score with the smallest element in the heap (which is always at the root due to the min-heap property). If the current passage’s score is higher than the smallest element:
Remove the smallest element from the heap.
Insert the current passage into the heap.

By doing this, we ensure that the heap always contains the (k) highest-scoring passages seen so far. This process is efficient because insertions and deletions in a min-heap take (O(\log k)) time, making the overall complexity of the algorithm (O(n \log k)), where (n) is the total number of passages.

#### Closing Summary

Using a min-heap of size (k), we can efficiently maintain the highest-scoring (k) passages seen so far with a single pass through all candidates. This approach leverages the heap’s ability to quickly manage and update the (k) largest elements, ensuring that our solution is both time and space efficient.

#### Common Mistakes/Follow-Ups
Incorrect Heap Initialization: Failing to initialize the heap correctly can lead to incorrect results.
Misunderstanding Heap Operations: Not understanding how to efficiently perform insertions and deletions in a min-heap can result in suboptimal performance.
Edge Cases: Failing to handle edge cases, such as when there are fewer than (k) passages or duplicate scores.

For example, if the input is:

CODE
Passages = [("passage1", 5), ("passage2", 9), ("passage3", 7), ("passage4", 6)]
k = 2

The output should be a list of the top (k) highest-scoring passages. Initially, the heap would have ([("passage1", 5), ("passage2", 9)]). After processing "passage3", it would replace "passage1" since its score is higher than 5. The final output for (k = 2) would be:

CODE
[("passage2", 9), ("passage3", 7)]

This approach ensures that we efficiently manage the (k) highest-scoring passages with a single pass through all candidates.

**Code:**

```python
import heapq

def top_k_passages(passages, k):
    # Initialize an empty min-heap
    heap = []

    for passage in passages:
        score, text = passage
        if len(heap) < k:
            heapq.heappush(heap, (score, text))
        else:
            if score > heap[0][0]:
                heapq.heapreplace(heap, (score, text))

    # Extract the top k elements from the heap and return them in sorted order by score
    return [x for x in reversed(sorted(heap))]

# Example input
passages = [("passage1", 5), ("passage2", 9), ("passage3", 7), ("passage4", 6)]
k = 2

top_k_passages(passages, k)
```

**Example input / output:**

```
Example input and output

passages = [("passage1", 5), ("passage2", 9), ("passage3", 7), ("passage4", 6)] k = 2

top_k_passages(passages, k)

CODE
Copy

Output:

[('passage2', 9), ('passage3', 7)]
```

**Flow:**

1. Yes
2. No
3. Start
4. Initialize min-heap
5. Next passage?
6. Insert or Replace in heapbased on score
7. Check if all passagesprocessed
8. Return top k elements fromheap
9. End

