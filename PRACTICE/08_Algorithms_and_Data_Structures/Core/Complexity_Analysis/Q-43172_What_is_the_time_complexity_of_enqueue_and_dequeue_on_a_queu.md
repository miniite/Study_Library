---
type: "practice"
difficulty: "Basic"
---

# What is the time complexity of enqueue and dequeue on a queue implemented with a linked list?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Linked Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Linked_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks for the time complexity of enqueue (add an element) and dequeue (remove an element) when a queue is implemented with a linked list. A queue is FIFO — First-In, First-Out — like a line at a ticket counter: you join at the back (tail) and leave from the front (head).

The interviewer is checking one specific insight: you must keep pointers to both ends (head and tail). If you only keep a head pointer, one of the two operations becomes O(n) because you'd have to walk the whole list to reach the other end. The whole point of this question is that with the right design, both operations are O(1). Let me define the design and justify each.

The design that makes it O(1)

Maintain two pointers on the linked list:

head → the front node (oldest element, the next one to leave).
tail → the back node (newest element, where the next one is added).

With both pointers available, neither operation ever traverses the list.

enqueue(x) — add to the tail → O(1):

Create a new node holding x.
Link the current tail's next to it: tail.next = newNode.
Move tail to the new node: tail = newNode. (Special case: if the queue was empty, set both head and tail to the new node.) All constant-time pointer operations → O(1).

dequeue() — remove from the head → O(1):

Read the value at head.
Advance: head = head.next.
If head becomes null, the queue is now empty, so set tail = null too. (Return the saved value; optionally free the old node.) All constant-time → O(1).

peek/front → O(1) as well (just read head.value).

Why this is O(1) — and the trap

We add at the tail and remove from the head. Because we hold a direct pointer to each end, we jump straight to where the work happens; the length of the list is irrelevant. There's no shifting (unlike an array where removing from the front shifts everything) and no searching.

The trap the question is really testing: choose which end to add and which to remove carefully, and keep a tail pointer.

Remove from head is naturally O(1) (you have head, and head.next is one step away).
Add to tail is O(1) only if you keep a tail pointer. Without one, enqueue must walk from head to the last node → O(n).
The reverse arrangement fails: if you tried to dequeue from the tail of a singly linked list, you'd need the node before tail to update the new tail, and finding it requires an O(n) walk (a singly linked list has no back-pointer). So the correct pairing is enqueue at tail, dequeue at head, with a tail pointer — that gives O(1)/O(1).

(A doubly linked list would let you remove from either end in O(1), but for a queue a singly linked list with head+tail pointers is enough and cheaper on memory.)

Comparison to array-based queues

This is why linked lists are attractive for queues. A naive array queue that dequeues by removing index 0 shifts all remaining elements → O(n) dequeue. Linked lists avoid that shift entirely. (A circular array / ring buffer also achieves O(1)/O(1) amortized, and array-backed can occasionally pay O(n) on resize — the linked list gives guaranteed, non-amortized O(1) per operation, at the cost of per-node pointer overhead and worse cache locality.)

Worked mental example

Start empty. enqueue(A), enqueue(B), enqueue(C):

head → A → B → C ← tail dequeue() → returns A (front), list = head → B → C ← tail enqueue(D) → head → B → C → D ← tail dequeue() → returns B. FIFO order out: A, B, ... — matches insertion order. Every operation touched only one end → constant work.
Summary — what a strong answer emphasizes
enqueue → O(1) and dequeue → O(1) for a linked-list queue.
The requirement: keep both head and tail pointers; enqueue at tail, dequeue at head.
Handle the empty-queue edge cases (first enqueue sets both pointers; last dequeue nulls the tail).
Space: O(n) for n elements, O(1) per operation. Linked lists give guaranteed O(1) (no amortization/resize) but cost extra pointer memory and have poorer cache locality than arrays.

#### Common mistakes
/ follow-up probes:

Forgetting the tail pointer → enqueue silently becomes O(n).
Trying to dequeue from the tail of a singly linked list → O(n) (no back pointer). Fix by choosing head for removal, or use a doubly linked list.
Forgetting to null tail when the queue empties → dangling pointer bug.

#### Follow-up
"Would a doubly linked list change the complexity?" → No, still O(1); it only adds the ability to remove from the tail in O(1), which a plain queue doesn't need.

**Comparison:**

```
Operation	Time	Requires	Reason
enqueue (add at tail)	O(1)	tail pointer	direct pointer to back; no walk
dequeue (remove at head)	O(1)	head pointer	direct pointer to front; no shift
peek (front)	O(1)	head pointer	read head value
space (total)	O(n)	—	one node + pointer per element
enqueue without tail pointer	O(n)	—	must traverse to last node
```

**Code:**

```python
class Node:
    __slots__ = ("val", "next")
    def __init__(self, val):
        self.val = val
        self.next = None

class LinkedQueue:
    def __init__(self):
        self.head = None
        self.tail = None

    def enqueue(self, x):            # O(1)
        node = Node(x)
        if self.tail is None:        # empty queue
            self.head = self.tail = node
        else:
            self.tail.next = node
            self.tail = node

    def dequeue(self):              # O(1)
        if self.head is None:
            raise IndexError("dequeue from empty queue")
        val = self.head.val
        self.head = self.head.next
        if self.head is None:        # became empty
            self.tail = None
        return val

    def peek(self):                 # O(1)
        if self.head is None:
            raise IndexError("peek from empty queue")
        return self.head.val
```

**Example input / output:**

```text
op          returns   list (head -> ... -> tail)
enqueue(A)  -         A
enqueue(B)  -         A -> B
enqueue(C)  -         A -> B -> C
dequeue()   A         B -> C
enqueue(D)  -         B -> C -> D
dequeue()   B         C -> D
(FIFO: elements leave in the same order they arrived; each op = O(1))
```

**Flow:**

1. enqueue(x):          dequeue():
2. new = Node(x)        val = head.val
3. if tail == null:     head = head.next
4. head=tail=new     if head == null:
5. else:                   tail = null
6. tail.next = new   return val
7. tail = new
8. [O(1)]              [O(1)]
9. head ─▶ [B] ─▶ [C] ─▶ [D] ◀─ tail
10. (dequeue here)        (enqueue here)

