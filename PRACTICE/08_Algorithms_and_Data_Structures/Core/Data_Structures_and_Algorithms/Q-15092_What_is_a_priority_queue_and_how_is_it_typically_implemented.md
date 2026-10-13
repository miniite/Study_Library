---
type: "practice"
difficulty: "Intermediate"
---

# What is a priority queue and how is it typically implemented?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queues and Stacks](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queues_and_Stacks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Define a priority queue (PQ) as an abstract data type (ADT), explain the operations it exposes, and then walk through how it is actually implemented in practice — primarily the binary heap, with a nod to the alternatives and why they exist.

What the interviewer is probing for: Do you separate the interface (what a PQ promises) from the implementation (how you make those promises fast)? Do you know the binary-heap invariant, the array-backed layout, the sift-up/sift-down mechanics, and the resulting complexities? Can you connect it to real usage (Dijkstra, scheduling, top-K, event simulation)? At the "intermediate experience" level they also want the trade-off discussion: why a heap and not a sorted array or a balanced BST, and when you'd reach for a different variant.

#### Assumptions
/clarifications I'd state: I'll assume a min-priority queue (smallest key = highest priority) unless told otherwise; a max-PQ is symmetric. I'll assume in-memory, single-threaded unless the interviewer wants a concurrent/distributed version. Keys are comparable.

Plan: ADT and its operations → the canonical binary-heap implementation (invariant, array mapping, insert, extract) → complexity table → alternatives and trade-offs → real-world uses and gotchas.

1. The ADT: what a priority queue promises

A priority queue is a collection where each element has an associated priority, and the defining rule is that you always remove the element with the highest priority next, regardless of insertion order. Contrast this with a plain queue (FIFO — remove the oldest) or a stack (LIFO — remove the newest). A PQ removes by priority, not by time.

Core operations:

push(x) / insert(x) — add an element with a priority.
peek() / top() — look at the highest-priority element without removing it.
pop() / extractMin() (or extractMax) — remove and return the highest-priority element.
size() / isEmpty().
Sometimes decreaseKey(x, newPriority) — raise an element's priority (crucial for Dijkstra/Prim).

The ADT says nothing about how this is stored — that's the implementer's job.

2. The canonical implementation: the binary heap

By far the most common implementation is the binary heap, a complete binary tree stored in a flat array. Two properties define it:

Shape (completeness): every level is fully filled except possibly the last, which fills left-to-right. This lets us store it in an array with no gaps.
Heap-order invariant: in a min-heap, every parent's key is ≤ both children's keys. So the minimum is always at the root (index 0). (Note: a heap is only partially ordered — siblings and cousins are unordered. That's exactly why insert/extract are logarithmic instead of linear.)

Array mapping (0-indexed): for node at index i:

parent = (i - 1) / 2
left child = 2i + 1
right child = 2i + 2

No pointers, great cache locality — this is why heaps are fast in practice, not just in Big-O.

push(x) — sift-up: append x at the end of the array (preserves completeness), then repeatedly swap it with its parent while it violates heap order (child < parent in a min-heap). It bubbles up at most log n levels → O(log n).

extractMin() — sift-down: the root is the answer. Move the last element to the root (keeps the tree complete), remove the old last slot, then repeatedly swap the new root with its smaller child until order is restored. At most log n levels → O(log n).

peek(): return array[0] → O(1).

Building a heap from n items (heapify): sift-down from the last internal node up to the root. Naively you'd guess O(n log n), but a tight analysis gives O(n) — most nodes are near the bottom and sift down only a level or two. Cheaper than n individual inserts.

3. Complexity and why the heap wins the default slot

The heap is the default because it gives O(log n) insert and O(log n) extract with O(1) peek and O(n) build, all in a tiny contiguous array. The alternatives each sacrifice one of those:

Unsorted array/list: insert O(1), but extract-min is O(n) (scan for the min). Fine only if you push a lot and rarely pop.
Sorted array: peek/extract O(1), but insert O(n) (shift to keep sorted). Fine only if you rarely push.
Balanced BST (red-black/AVL) or skip list: all ops O(log n) and you get ordered iteration, predecessor/successor, and arbitrary delete. But higher constant factors and pointer-chasing (worse cache behavior) than a heap, and more code. Use it when you need more than PQ semantics (e.g., range queries).
Binary heap: the sweet spot for pure PQ semantics — smallest constants, array-backed.
d-ary heap: each node has d children; shallower tree → faster decreaseKey/insert (fewer levels) at the cost of slower extract (comparing d children). Tunable for decrease-key-heavy loads.
Fibonacci / pairing heap: amortized O(1) insert and decreaseKey, O(log n) extract. Theoretically optimal for Dijkstra (O(E + V log V)), but big constants and complexity mean binary heaps usually win in practice.
4. Real-world uses and gotchas
Dijkstra / Prim: the PQ orders vertices/edges by tentative distance; decreaseKey (or the lazy "push a new pair, skip stale pops") relaxes edges.
Task/OS scheduling, event-driven simulation: next event = smallest timestamp.
Top-K / streaming: keep a size-K min-heap of the K largest seen so far — O(n log K), O(K) memory. This is a classic follow-up.
Median-of-a-stream: two heaps (a max-heap of the low half, a min-heap of the high half).
Huffman coding, A*, load balancing (least-loaded server).

Practical gotchas an interviewer likes to hear:

Ties: heaps aren't stable. If FIFO-among-equal-priority matters, add a monotonic insertion counter to the key so equal priorities break ties by arrival.
decreaseKey needs a handle: a plain binary heap can't find an arbitrary element in O(1). Either keep an index map (element → position) updated on every swap, or use the lazy deletion trick: push the updated entry and discard stale ones when popped.
Off-the-shelf: Java PriorityQueue (min-heap, pass a Comparator for max/custom), Python heapq (min-heap on a list — negate keys or use tuples for max), C++ std::priority_queue (max-heap by default).
Summary — what a strong answer emphasizes

A priority queue is an ADT ("always remove the highest-priority element"), and the binary heap is its default implementation: a complete binary tree in an array, kept partially ordered by the heap invariant, giving O(1) peek, O(log n) push, O(log n) pop, O(n) build. Nail the sift-up/sift-down mechanics and the parent/child index arithmetic, then show judgment by comparing against sorted arrays and balanced BSTs and naming when d-ary/Fibonacci/pairing heaps or a BST are the better call. Mention ties, decreaseKey handles, and a concrete use (Dijkstra or top-K) to land it.

#### Common mistakes
conflating the ADT with the heap; claiming a heap is fully sorted; forgetting that build-heap is O(n); saying decreaseKey is O(1) on a plain binary heap without a position map; missing tie-stability.

**Comparison:**

```
Implementation	insert	peek	extractMin	decreaseKey	build (n items)	Notes
Unsorted array	O(1)	O(n)	O(n)	O(1)*	O(n)	scan to find min
Sorted array	O(n)	O(1)	O(1)	O(n)	O(n log n)	shifting is the cost
Binary heap	O(log n)	O(1)	O(log n)	O(log n)	O(n)	array-backed, best constants — the default
d-ary heap	O(log_d n)	O(1)	O(d·log_d n)	O(log_d n)	O(n)	shallower; tune for decrease-key load
Balanced BST	O(log n)	O(log n)	O(log n)	O(log n)	O(n log n)	ordered iteration/range as bonus
Fibonacci heap	O(1)†	O(1)	O(log n)†	O(1)†	O(n)	†amortized; big constants, rarely worth it

* with a position map. † amortized.

Concept	Queue (FIFO)	Stack (LIFO)	Priority queue
Removal rule	oldest first	newest first	highest priority first
Typical impl	ring buffer / linked list	array / linked list	binary heap
```

**Code:**

```
Min binary heap backed by a Python list (illustrative; stdlib heapq does this in C)

class MinHeap: def init(self): self.a = []

CODE
Copy
def peek(self):
    if not self.a:
        raise IndexError("empty heap")
    return self.a[0]                     # O(1)

def push(self, x):                       # O(log n)
    self.a.append(x)
    self._sift_up(len(self.a) - 1)

def pop(self):                           # O(log n) — extractMin
    if not self.a:
        raise IndexError("empty heap")
    root = self.a[0]
    last = self.a.pop()                  # remove last slot -> stays complete
    if self.a:
        self.a[0] = last
        self._sift_down(0)
    return root

def _sift_up(self, i):
    while i > 0:
        parent = (i - 1) // 2
        if self.a[i] < self.a[parent]:   # min-heap: child must not be < parent
            self.a[i], self.a[parent] = self.a[parent], self.a[i]
            i = parent
        else:
            break

def _sift_down(self, i):
    n = len(self.a)
    while True:
        l, r, smallest = 2 * i + 1, 2 * i + 2, i
        if l < n and self.a[l] < self.a[smallest]:
            smallest = l
        if r < n and self.a[r] < self.a[smallest]:
            smallest = r
        if smallest == i:
            break
        self.a[i], self.a[smallest] = self.a[smallest], self.a[i]
        i = smallest

@staticmethod
def heapify(items):                      # build in O(n)
    h = MinHeap()
    h.a = list(items)
    for i in range(len(h.a) // 2 - 1, -1, -1):
        h._sift_down(i)
    return h
CODE
Copy

```python
# Idiomatic real-world usage with the standard library
import heapq
pq = []
heapq.heappush(pq, (2, "medium"))     # (priority, payload) tuples order by priority
heapq.heappush(pq, (1, "urgent"))
heapq.heappush(pq, (3, "low"))
print(heapq.heappop(pq))              # (1, 'urgent') -> smallest priority first

# Top-K largest with a size-K MIN-heap: O(n log K)
def top_k(nums, k):
    h = []
    for x in nums:
        heapq.heappush(h, x)
        if len(h) > k:
            heapq.heappop(h)          # drop current smallest
    return sorted(h, reverse=True)
```

**Example input / output:**

```

Dry-run of a min-heap. Array shown as the flat backing store (index 0 = root = current min).

CODE
Copy
push(5)   -> [5]
push(3)   -> append 3 -> [5,3]; sift-up: 3<5 swap -> [3,5]
push(8)   -> append 8 -> [3,5,8]; 8>3 stop        -> [3,5,8]
push(1)   -> append 1 -> [3,5,8,1]; sift-up:
             idx3 parent idx1 -> 1<5 swap -> [3,1,8,5]
             idx1 parent idx0 -> 1<3 swap -> [1,3,8,5]
peek()    -> 1

pop()     -> return 1. Move last(5) to root: [5,3,8]; sift-down:
             children of idx0 = 3(idx1),8(idx2); smallest=3; 5>3 swap -> [3,5,8]
             -> [3,5,8]
pop()     -> return 3. Move last(8) to root: [8,5]; sift-down:
             child 5<8 swap -> [5,8]
             -> [5,8]

top_k([4,1,7,3,9,2], k=3):

CODE
Copy
push4 -> [4]
push1 -> [1,4]
push7 -> [1,4,7]
push3 -> [1,3,4,7] size 4>3 -> pop 1 -> [3,4,7]
push9 -> [3,4,7,9] size 4>3 -> pop 3 -> [4,7,9]
push2 -> [2,4,7,9] size 4>3 -> pop 2 -> [4,7,9]
result sorted desc -> [9, 7, 4]   # the 3 largest
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. push(x)
6. append x at end (keeps treecomplete)
7. x < parent? (min-heap)
8. swap with parent; move up
9. done — O(log n)
10. pop() / extractMin
11. save root = array[0] (the min)
12. move LAST element to root;shrink array
13. root > smaller child?
14. swap with smaller child; movedown
15. return saved root — O(log n)

