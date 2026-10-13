---
type: "practice"
difficulty: "Intermediate"
---

# What is a heap, and what is the difference between a min-heap and a max-heap?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Algorithms](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting_Algorithms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what's really being asked

This is a fundamentals question with a comparison built in: (1) define what a heap is as a data structure, and (2) crisply distinguish a min-heap from a max-heap. At the "intermediate experience" level the interviewer expects more than a textbook definition — they want you to explain the heap property, the array-backed representation, the core operations and their complexities, and why heaps matter (priority queues, top-k, scheduling, Dijkstra). The min vs max distinction should be answered in one clean sentence, then reinforced with the operations each makes cheap.

No real clarifying questions are needed here; it's a definitional question. I'll define it precisely, show the array layout that makes it efficient, walk the two invariant-restoring operations (sift-up, sift-down), give complexities, and close with the min/max contrast and where each is used.

What a heap is

A heap is a complete binary tree that satisfies the heap property. "Complete" means every level is fully filled except possibly the last, which is filled left-to-right — there are no gaps. This completeness is what lets a heap be stored compactly in a flat array with no pointers.

The heap property is an ordering invariant between each parent and its children (not across siblings, and not a full sort):

Min-heap: every parent is ≤ both of its children. Consequence: the minimum element of the whole heap sits at the root (index 0).
Max-heap: every parent is ≥ both of its children. Consequence: the maximum element sits at the root.

Crucially, a heap is only partially ordered. Knowing the root is the min (or max) tells you nothing about the relative order of the rest — the left subtree is not necessarily smaller than the right. That's the difference between a heap and a sorted array, and it's exactly why heaps are cheaper to maintain than a fully sorted structure.

The array representation (why heaps are fast and pointer-free)

Because the tree is complete, we can lay it out in an array by breadth-first order. For a node at index i (0-based):

left child = 2*i + 1
right child = 2*i + 2
parent = (i - 1) / 2 (integer division)

No node objects, no child pointers — just index arithmetic. This gives excellent cache locality and O(1) navigation.

The two core operations

Insertions and deletions can each temporarily break the heap property; we fix it by "bubbling" an element along a single root-to-leaf path.

Insert (push): put the new element at the end of the array (next open leaf, preserving completeness), then sift-up: while it violates the property with its parent, swap it upward. Path length is the tree height → O(log n).
Extract-min / extract-max (pop): the answer is at the root. Move the last element into the root slot, shrink the array by one, then sift-down: repeatedly swap the root with its smaller child (min-heap) or larger child (max-heap) until the property holds again. Again one root-to-leaf path → O(log n).
Peek (find-min / find-max): just read index 0 → O(1).
Build-heap from an arbitrary array: sift-down from the last internal node up to the root. A clever amortized analysis shows this is O(n), not O(n log n) — most nodes are near the bottom and sift down only a little.

Height of a complete tree of n nodes is ⌊log₂ n⌋, so every path-based operation is logarithmic.

Min-heap vs max-heap — the actual difference

The two are structurally identical; the only difference is the direction of the comparison in the heap property (and therefore in sift-up/sift-down):

Min-heap uses parent ≤ child → cheap access to the smallest.
Max-heap uses parent ≥ child → cheap access to the largest.

You can turn one into the other trivially: to simulate a max-heap using a min-heap library, insert negated values (or invert the comparator). In Java, PriorityQueue is a min-heap by default; new PriorityQueue<>(Collections.reverseOrder()) makes it a max-heap. In Python, heapq is a min-heap only — for a max-heap you push -x or use tuples with negated keys.

Where heaps are used (why the interviewer cares)
Priority queues — the canonical use. Always pull the highest-priority (or lowest-cost) item next.
Top-k / k-th largest — a size-k min-heap tracks the k largest seen so far in O(n log k).
Dijkstra / Prim (graph shortest-path / MST) — a min-heap picks the next closest vertex.
Heapsort — build a heap then repeatedly extract → O(n log n), in-place, not stable.
Median maintenance / streaming stats — a max-heap (lower half) + min-heap (upper half) gives O(log n) median updates.
Event schedulers, timers, load balancers — pull the next event by time or least-loaded server.
Complexity summary
Operation	Complexity
Peek min/max (root)	O(1)
Insert (push)	O(log n)
Extract min/max (pop)	O(log n)
Build heap from array	O(n)
Search for arbitrary element	O(n) — heap is not built for lookup
Space	O(n)

#### Common mistakes
& follow-up probes
Saying a heap is fully sorted — it is only partially ordered; only the root is guaranteed extremal.
Confusing this heap with the memory "heap" — the runtime memory region is unrelated; same word, different concept.
Claiming build-heap is O(n log n) — the tight bound is O(n); be ready to justify with the "most nodes are shallow" argument.
"Find an arbitrary element quickly?" — no, that's O(n); use a hash map alongside if you need decrease-key by value (as Dijkstra implementations often do).

#### Follow-up
"min-heap or max-heap for k-th largest?" — a min-heap of size k (see the related question); for k-th smallest you'd use a max-heap of size k. Knowing this inversion crisply is the mark of comfort with heaps.
Summary — what a strong answer emphasizes

A heap is a complete binary tree with the parent-child heap property, stored in an array via index arithmetic, giving O(1) peek and O(log n) insert/extract. Min-heap = parent ≤ children → min at root; max-heap = parent ≥ children → max at root — identical structure, opposite comparator. Reinforce with the real uses (priority queue, top-k, Dijkstra, heapsort) and the O(n) build-heap fact.

**Comparison:**

```
Aspect	Min-heap	Max-heap
Heap property	parent ≤ children	parent ≥ children
Element at root	Minimum	Maximum
O(1) peek gives	Smallest	Largest
Typical use	Dijkstra, k-th largest (size-k), min priority queue	Median (lower half), k-th smallest (size-k), max priority queue
Java default	new PriorityQueue<>()	PriorityQueue<>(Collections.reverseOrder())
Python heapq	native	push -x (negate)
Operation	Complexity	Note
peek	O(1)	read root
insert / extract	O(log n)	one root-to-leaf path
build-heap	O(n)	amortized, not O(n log n)
arbitrary search	O(n)	heaps aren't for lookup
```

**Code:**

```

// Min-heap and max-heap in Java via PriorityQueue PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // min-heap (default) PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder()); // max-heap

minHeap.offer(5); minHeap.offer(1); minHeap.offer(3); minHeap.peek(); // 1 (smallest at root, O(1)) minHeap.poll(); // 1 (extract-min, O(log n))

maxHeap.offer(5); maxHeap.offer(1); maxHeap.offer(3); maxHeap.peek(); // 5 (largest at root)

CODE
Copy

```python
# Python heapq is a MIN-heap only; negate for a max-heap
import heapq
h = []
heapq.heappush(h, 5); heapq.heappush(h, 1); heapq.heappush(h, 3)
heapq.heappop(h)          # -> 1  (min)

maxh = []
for x in (5, 1, 3):
    heapq.heappush(maxh, -x)   # push negatives
-heapq.heappop(maxh)      # -> 5  (max, remember to negate back)
TEXT
Copy
// The sift-down invariant restore used by extract (min-heap version), pseudocode
siftDown(i):
    while true:
        l = 2i+1; r = 2i+2; smallest = i
        if l < n and a[l] < a[smallest]: smallest = l
        if r < n and a[r] < a[smallest]: smallest = r
        if smallest == i: break
        swap(a[i], a[smallest]); i = smallest
```

**Example input / output:**

```

Min-heap built from [5, 3, 8, 1, 9, 2], shown as array + tree:

CODE
Copy
Insert order: 5,3,8,1,9,2  ->  valid min-heap array: [1, 3, 2, 5, 9, 8]

Tree view (index in brackets):
            1[0]
          /      \
       3[1]       2[2]
      /    \      /
   5[3]   9[4]  8[5]

parent(i) = (i-1)/2 ; left(i) = 2i+1 ; right(i) = 2i+2
Root = 1 = global minimum. peek() -> 1
poll() -> returns 1, moves 8 to root, sifts down -> new heap [2, 3, 8, 5, 9]

Max-heap of the same set: root would be 9, e.g. [9, 5, 8, 1, 3, 2], peek() -> 9.
```

**Flow:**

1. Max-heap: parent >= children
2. 9
3. 5
4. 8
5. 1
6. 3
7. 2
8. Min-heap: parent <= children
9. 1
10. 3
11. 2
12. 5
13. 9
14. 8

