---
type: "practice"
difficulty: "Intermediate"
---

# When would you use a deque over a list, and why is popleft O(1) on a deque?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Membership and Collection Complexity](<../../../../TOOLS/Python/Python_Language/Membership_and_Collection_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

Two linked questions: (1) when do I reach for collections.deque instead of a plain list, and (2) why is deque.popleft() O(1) while the equivalent on a list is O(n). The interviewer is probing whether I understand the underlying data structures — that a Python list is a contiguous dynamic array and a deque is a doubly-linked list of fixed-size blocks — and can connect that memory layout to Big-O of front/back operations. This is a classic "do you know why, not just what" question. A strong intermediate answer states the cost table, explains the array-shift cause, gives real use cases (queues, BFS, sliding windows, bounded buffers), and notes the trade-off deque pays in return (no cheap random indexing).

Plan: contrast the two structures' internals → derive the cost of each end operation → give the concrete "when to use deque" scenarios → state the trade-offs deque makes → summarize with the mental model.

The internals — where the difference comes from

A Python list is a dynamic array: it stores its elements in one contiguous block of memory (an array of pointers). Random access lst[i] is O(1) because the address is just base + i * pointer_size. Appending at the end (append) is amortized O(1) — the array is over-allocated so most appends just write into spare slots, with an occasional O(n) resize. But inserting or removing at the front (insert(0, x), pop(0)) is O(n): every one of the remaining elements has to be physically shifted one slot to fill or open the gap, because the array must stay contiguous.

A deque (double-ended queue) is implemented as a doubly-linked list of fixed-size blocks (arrays of ~64 slots linked together). It keeps pointers to both the leftmost and rightmost block/position. Because it tracks both ends and the blocks are linked, adding or removing at either end is O(1): you touch only the boundary block and, when a block fills or empties, splice a block in or out — no bulk shifting of the other elements.

Why popleft() is O(1) on a deque but pop(0) is O(n) on a list

deque.popleft() removes the element at the head. The deque already holds a pointer to the left end, so it reads that slot, advances the head pointer by one (and unlinks the block if it just emptied), and returns — a constant number of pointer updates regardless of how many elements remain. O(1).

list.pop(0) removes index 0 of a contiguous array. To keep indices contiguous and 0-based, every remaining element [1..n-1] must be copied down by one position (a memmove of n-1 pointers). That work grows linearly with the list length. O(n). The symmetric case — list.insert(0, x) — is O(n) for the same reason (shift everything right). So a list used as a FIFO queue by append + pop(0) degrades to O(n) per dequeue and O(n²) overall for n items, which is the exact anti-pattern deque exists to fix.

Both ends of a deque are symmetric: append/pop (right) and appendleft/popleft (left) are all O(1).

When to use a deque over a list
FIFO queues — anything where you push at one end and pop at the other. Task queues, producer/consumer buffers, rate-limiter windows. list.pop(0) is the trap here.
BFS / level-order traversal / topological sort — the frontier is a queue; you append neighbors and popleft the next node. Using a list would make graph BFS quadratic.
Sliding-window algorithms (e.g. "max in each window of size k") — a monotonic deque pops stale indices from the left and small values from the right, both O(1), giving an O(n) solution.
Bounded / rolling buffers — deque(maxlen=N) auto-evicts from the opposite end when full: perfect for "keep the last N log lines / prices / events" without manual trimming. This is a feature a list simply doesn't have.
Undo stacks or double-ended work-stealing where you legitimately push and pop from both ends.
The trade-off — what deque gives up

Deque is not a strict upgrade; it's a different shape with different costs:

Random access is O(n) on a deque. dq[i] for a middle index has to walk block by block from the nearest end. A list gives O(1) lst[i]. So if your workload is index-heavy (random reads/writes, binary search, sorting), use a list.
No slicing (dq[2:5] raises TypeError) and no .sort() in place; you'd convert to a list first.
Middle insertion/removal is O(n) for both, so neither wins there.
Memory: deque has some per-block overhead but avoids the big contiguous reallocation spikes lists have; for append-heavy front/back workloads it's usually comparable or better.

Rule of thumb: need cheap ends → deque; need cheap indexing/slicing/sort → list. If you find yourself calling list.pop(0) or list.insert(0, …) in a loop, that's the signal to switch to a deque.

Summary

A list is a contiguous dynamic array: O(1) index and amortized-O(1) append, but O(n) at the front because elements must shift. A deque is a doubly-linked list of blocks tracking both ends, so append, appendleft, pop, and popleft are all O(1) — popleft just advances a head pointer with no shifting, which is exactly why it beats list.pop(0)'s O(n). Reach for deque for queues, BFS, sliding windows, and bounded buffers (maxlen); stay with list when you need O(1) random access, slicing, or sorting. The interviewer wants the array-shift-vs-pointer-advance explanation — that's the crux. Follow-up probes: "amortized append on a list?", "what's maxlen do?", and "why is dq[i] not O(1)?".

**Comparison:**

```
Operation	list	deque
index x[i] (random access)	O(1)	O(n)
append at right	amortized O(1)	O(1)
pop at right	O(1)	O(1)
insert at left insert(0,x) / appendleft	O(n)	O(1)
pop at left pop(0) / popleft	O(n)	O(1)
middle insert/remove	O(n)	O(n)
slicing x[a:b]	O(k)	not supported
bounded auto-eviction	manual	built-in via maxlen
```

**Code:**

```python
from collections import deque

# --- deque as an O(1) FIFO queue ---
q = deque()
q.append("a"); q.append("b"); q.append("c")   # push right, O(1)
first = q.popleft()          # "a", removed from the front in O(1)
# q -> deque(['b', 'c'])

# --- BFS uses a deque frontier ---
def bfs(graph, start):
    seen, frontier, order = {start}, deque([start]), []
    while frontier:
        node = frontier.popleft()     # O(1) dequeue
        order.append(node)
        for nxt in graph[node]:
            if nxt not in seen:
                seen.add(nxt)
                frontier.append(nxt)  # O(1) enqueue
    return order

# --- bounded rolling buffer: keep only the last 3 items ---
recent = deque(maxlen=3)
for x in [1, 2, 3, 4, 5]:
    recent.append(x)          # auto-evicts from the left when full
# recent -> deque([3, 4, 5], maxlen=3)

# --- the anti-pattern deque replaces ---
lst = [1, 2, 3, 4]
lst.pop(0)                    # O(n): shifts 2,3,4 down one slot each
```

**Example input / output:**

```

Queue trace:

CODE
Copy
append "a","b","c"  -> deque(['a','b','c'])
popleft()           -> returns 'a',  deque(['b','c'])   # O(1)

BFS on graph {A:[B,C], B:[D], C:[D], D:[]} from A:

CODE
Copy
bfs(graph, 'A') -> ['A', 'B', 'C', 'D']

Bounded buffer maxlen=3, feeding 1..5:

CODE
Copy
after 1        -> [1]
after 1,2,3    -> [1,2,3]
after 4        -> [2,3,4]   # 1 evicted from left
after 5        -> [3,4,5]   # 2 evicted from left

List vs deque front-removal cost:

CODE
Copy
list.pop(0)     -> O(n)  (memmove of remaining n-1 elements)
deque.popleft() -> O(1)  (advance head pointer)
```

**Flow:**

1. deque.popleft() — O(1)
2. head->[a]<->[b]<->[c]<->[d]<-tail
3. read a, advance head pointeronly
4. head->[b]<->[c]<->[d]<-tail
5. list.pop(0) — O(n)
6. [a][b][c][d]
7. remove a, then SHIFT b,c,d leftone slot each
8. [b][c][d][ ]

