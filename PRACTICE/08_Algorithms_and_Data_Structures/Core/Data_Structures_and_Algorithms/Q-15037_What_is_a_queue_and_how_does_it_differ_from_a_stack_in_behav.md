---
type: "practice"
difficulty: "Basic"
---

# What is a queue and how does it differ from a stack in behavior?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queues and Stacks](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queues_and_Stacks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

Two things: (1) define a queue with its core operations, and (2) explain how its behavior differs from a stack. The interviewer wants to see that I understand FIFO vs LIFO, know the operations and their O(1) cost, and can articulate when each is appropriate — not just recite acronyms. The comparison is the heart of the question, so I'll define the queue cleanly and then contrast it against the stack along ordering, access points, and use cases.

#### Assumptions
/clarifications: I'll treat "queue" as the classic FIFO abstract data type. I'll mention common variants (circular, deque, priority queue) briefly since they're natural follow-ups, but the core answer is FIFO. Backing store can be an array (circular buffer) or a linked list.

Definition — from first principles

A queue is a linear collection that follows FIFO: First In, First Out. The first element added is the first one removed — exactly like a line of people at a checkout: you join at the rear (back) and you're served from the front. Two ends are in play, and each is used for a different purpose.

Core operations, all O(1):

enqueue(x) — add x at the rear.
dequeue() — remove and return the element at the front.
front() / peek() — read the front element without removing it.
isEmpty() — whether the queue has any elements.
(often) size().

#### Implementation
notes: a naive array where you dequeue by shifting everything left is O(n) per dequeue — bad. The right implementations are a circular buffer (front/rear indices wrap around modulo capacity, O(1) each) or a singly linked list with head and tail pointers (dequeue from head, enqueue at tail, both O(1)). Dequeuing an empty queue is an error (underflow); a fixed-capacity queue can overflow.

How a queue differs from a stack — behavior

The single defining difference is the order in which elements come out:

Stack = LIFO. You push and pop at one end (the top). The most recently added element leaves first. Model it as "reverse/unwind": undo, recursion/call stack, DFS, matching brackets.
Queue = FIFO. You enqueue at the rear and dequeue from the front — two different ends. The earliest added element leaves first. Model it as "process in arrival order / fairness": task scheduling, print jobs, BFS, producer–consumer buffers, request handling.

A concrete way to feel the difference: push/enqueue 1, 2, 3 into each, then remove all three.

Stack pops: 3, 2, 1 (reversed).
Queue dequeues: 1, 2, 3 (preserved).

So the choice is really: do you need to reverse the order of processing (stack) or preserve it (queue)? That single question decides which structure fits. Both give O(1) core operations; the difference is purely the access discipline, not the cost.

Where each is used (why the behavior matters)
Queue (FIFO): BFS/level-order traversal of trees and graphs (visit nearest nodes first); CPU/task scheduling; message queues and request buffers between a fast producer and slower consumer; print spoolers; rate limiting. All need fairness / arrival order.
Stack (LIFO): recursion and the call stack; expression evaluation and syntax/bracket checking; undo/redo; browser back button; DFS. All need reverse/most-recent-first processing.
Variants worth naming (follow-ups)
Circular queue — reuses freed slots at the front via modulo indexing; avoids the O(n) shift.
Deque (double-ended queue) — insert/remove at both ends; a superset that can act as either a stack or a queue.
Priority queue — dequeues by priority, not arrival order; typically a binary heap with O(log n) enqueue/dequeue (not FIFO, so technically a different ADT).
Summary — what a strong answer emphasizes
Queue = FIFO, two ends (enqueue at rear, dequeue at front), O(1) operations, implemented as a circular buffer or head/tail linked list.
The key contrast: queue preserves order (FIFO), stack reverses it (LIFO); stack uses one end, queue uses two.
Tie the behavior to use cases: BFS / scheduling / buffering for queues vs recursion / undo / DFS / bracket matching for stacks.
Mention the O(n)-dequeue pitfall of a naive array queue and the circular-buffer fix.

#### Common mistakes
saying "queue is the opposite of a stack" without stating FIFO vs LIFO and the two-ends-vs-one-end distinction; implementing dequeue with an O(n) array shift; confusing a priority queue (heap) with a plain FIFO queue.

**Comparison:**

```
Aspect	Queue	Stack
Order	FIFO (first in, first out)	LIFO (last in, first out)
Insert / remove ends	insert at rear, remove at front (two ends)	both at the top (one end)
Core ops	enqueue, dequeue, front	push, pop, peek
Op cost	O(1)	O(1)
Removal of 1,2,3	1, 2, 3	3, 2, 1
Typical uses	BFS, scheduling, buffering, message queues	recursion, undo/redo, DFS, bracket matching
Good backing store	circular buffer / head+tail linked list	dynamic array / linked list
```

**Code:**

```

from collections import deque

collections.deque gives O(1) enqueue/dequeue at both ends.

class Queue: def init(self): self._d = deque() def enqueue(self, x): self._d.append(x) # add at rear def dequeue(self): return self._d.popleft() # remove from front (raises if empty) def front(self): return self._d[0] def is_empty(self): return not self._d def size(self): return len(self._d)

CODE
Copy

```python
# Side-by-side behavior: same inserts, opposite removal order.
stack, queue = [], deque()
for x in (1, 2, 3):
    stack.append(x)      # push
    queue.append(x)      # enqueue

stack_out = [stack.pop()     for _ in range(3)]   # -> [3, 2, 1]  (LIFO)
queue_out = [queue.popleft() for _ in range(3)]   # -> [1, 2, 3]  (FIFO)
```

**Example input / output:**

```text
Insert order into both: 1, 2, 3

Stack removal (pop):      3, 2, 1     # last in, first out
Queue removal (dequeue):  1, 2, 3     # first in, first out

Queue edge cases:
  enqueue(5); front() -> 5 ; dequeue() -> 5 ; is_empty() -> True
  dequeue() on empty  -> error (underflow)
```

**Flow:**

1. Queue [FIFO]
2. rear
3. ... elements ...
4. front
5. enqueue
6. dequeue

