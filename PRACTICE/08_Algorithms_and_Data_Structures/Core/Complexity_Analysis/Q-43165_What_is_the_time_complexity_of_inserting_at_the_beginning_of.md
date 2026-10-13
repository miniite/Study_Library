---
type: "practice"
difficulty: "Basic"
---

# What is the time complexity of inserting at the beginning of an array vs a linked list?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Set Operation Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Set_Operation_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Compare the time complexity of inserting a new element at the beginning (index 0 / the head) of an array versus a singly/doubly linked list. The interviewer wants the two Big-O numbers and, more importantly, the reason each one is what it is — rooted in how the two data structures are laid out in memory.

What the interviewer is probing (basic concept, fresher-level): Do I know the fundamental difference between contiguous storage (array) and node-with-pointers storage (linked list), and can I explain why prepending is cheap in one and expensive in the other? I should define terms, not hand-wave.

#### Assumptions
/clarifications: "Array" = contiguous array (or a dynamic array like ArrayList/std::vector/Python list). "Insert at beginning" = make the new element the first one, keeping all existing elements. For the linked list, I assume we already hold a reference to the head (which is normal). I'll give the answer for both structures and explain the mechanism.

Plan: State the two complexities → explain the array case (why shifting is needed) → explain the linked-list case (why only pointers change) → note the important caveat about finding a position vs inserting there → summarize with a comparison table.

The short answer
Array (insert at beginning): O(n) — every existing element must shift one slot to the right.
Linked list (insert at beginning / prepend at head): O(1) — just create a node and rewire a couple of pointers.

That O(n)-vs-O(1) gap is the textbook reason people reach for a linked list when they frequently add/remove at the front.

Why inserting at the front of an array is O(n)

An array stores its elements in one contiguous block of memory, at consecutive indices 0, 1, 2, …, n−1. Index i lives at a fixed offset from the start, which is exactly why arrays give O(1) random access.

But that same rigidity makes prepending expensive. Index 0 is already occupied, and the new element must end up at index 0 while everything else keeps its relative order. So we have to move every existing element one position to the right — element at index n−1 goes to n, n−2 goes to n−1, …, index 0 goes to index 1 — and only then write the new value into slot 0. That's n moves for n existing elements → O(n) time.

For a dynamic array there's a second possible cost: if the backing array is full, it must also resize (allocate a bigger array and copy). Resizing is O(n) too but happens rarely, so it's O(1) amortized and doesn't change the prepend story — the shifting alone already makes front-insertion O(n).

(Contrast: appending at the end of a dynamic array is O(1) amortized, because no shifting is needed — you just write past the last element.)

Why inserting at the front of a linked list is O(1)

A linked list stores each element in a separate node, and each node holds the value plus a pointer (reference) to the next node. The nodes can sit anywhere in memory; the list's order is defined by the pointer chain, not by physical position. The list keeps a reference to the head (first node).

To insert at the beginning:

Create a new node holding the value.
Set the new node's next pointer to the current head.
Update head to point to the new node.

That's a constant number of operations — no matter whether the list has 5 elements or 5 million, you touch the same two or three pointers. Nothing shifts. So front insertion is O(1). (In a doubly linked list you also set one prev pointer — still O(1).)

The important caveat: finding a spot vs inserting at a known spot

A fresher-favorite trap: the O(1) linked-list advantage applies to inserting at a position you already have a reference to — like the head. If you need to insert in the middle (say, at index k), the linked list must first walk from the head k steps to reach that position, which is O(n) traversal, even though the pointer rewire itself is O(1). An array can reach index k in O(1) (random access) but then still pays O(n) to shift. So for the front specifically — where the linked list already has the head reference — the linked list wins cleanly at O(1). This is exactly why a linked list is a natural fit for stacks/queues (front/end operations) and an array is a natural fit for random access and end-appends.

Summary — what a strong answer emphasizes
Array front insert = O(n) because contiguous storage forces shifting all n elements right; you gain O(1) random access but pay for prepend.
Linked list front insert = O(1) because you only allocate a node and rewire the head pointer; order lives in pointers, so nothing moves.
Root cause = memory layout: contiguous (array) vs node-and-pointer (linked list).
Caveat: linked-list O(1) holds only when you already have a reference to the insertion point (head). Inserting elsewhere costs O(n) to find the node first.

#### Common mistakes
saying array prepend is O(1) (confusing it with append-at-end, which is the O(1)/amortized case); claiming linked-list insert is O(1) everywhere (it's O(n) to locate a middle position first); forgetting that the array's cost is the shifting, not the writing.

**Comparison:**

```
Operation: insert at beginning	Array / Dynamic array	Linked list (at head)
Time complexity	O(n)	O(1)
Why	Must shift all n elements right	Only allocate node + rewire head pointer
Memory layout	Contiguous block, fixed offsets	Separate nodes joined by pointers
Extra cost	Possible resize (O(n), rare → O(1) amortized)	One node allocation
Random access to index i	O(1)	O(n) (must traverse)
Insert at END	O(1) amortized (no shift)	O(1) if tail kept, else O(n) to find tail
```

**Code:**

```python
# ARRAY (Python list): insert at index 0 -> shifts everything right -> O(n)
arr = [10, 20, 30]
arr.insert(0, 5)      # internally moves 10,20,30 right by one, then writes 5
print(arr)            # [5, 10, 20, 30]  -- O(n)

# LINKED LIST: prepend at head -> just rewire pointers -> O(1)
class Node:
    def __init__(self, val, nxt=None):
        self.val = val
        self.next = nxt

class LinkedList:
    def __init__(self):
        self.head = None

    def push_front(self, val):        # insert at beginning: O(1)
        self.head = Node(val, self.head)   # new node points to old head; head = new node

    def to_list(self):
        out, cur = [], self.head
        while cur:
            out.append(cur.val)
            cur = cur.next
        return out

ll = LinkedList()
for v in (30, 20, 10):
    ll.push_front(v)      # each is O(1), regardless of list size
print(ll.to_list())       # [10, 20, 30]
```

**Example input / output:**

```text
Array insert at front:
  input : arr = [10, 20, 30], insert 5 at index 0
  action: shift 30->idx3, 20->idx2, 10->idx1, write 5 at idx0   (3 shifts = O(n))
  output: [5, 10, 20, 30]

Linked list prepend:
  input : list head -> 10 -> 20 -> 30, push_front(5)
  action: newNode(5).next = head(10); head = newNode(5)         (2 pointer ops = O(1))
  output: 5 -> 10 -> 20 -> 30
```

**Flow:**

1. INSERT 5 AT THE BEGINNING
2. Array (O(n)) — everything shifts right:
3. before: [10][20][30]
4. \   \   \   (each moves one slot right)
5. after : [ 5][10][20][30]
6. Linked list (O(1)) — only pointers change:
7. before:            head -> (10) -> (20) -> (30) -> null
8. step 1: new (5).next = head(10)
9. step 2: head = (5)
10. after :  head -> ( 5) -> (10) -> (20) -> (30) -> null

