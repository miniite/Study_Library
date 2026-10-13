---
type: "practice"
difficulty: "Basic"
---

# Explain what a doubly linked list is and one advantage it has over a singly linked list.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List Techniques](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List_Techniques.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Define a doubly linked list (DLL) clearly, and name at least one concrete advantage it has over a singly linked list (SLL). This is a basic-concept question, so the interviewer wants a precise, correct definition, evidence that you understand how the pointers work, and one well-explained advantage — not a laundry list of half-remembered facts. The bar is clarity, not cleverness.

Plan: Define a node and the list, contrast it visually with an SLL, give the headline advantage (bidirectional traversal → O(1) deletion when you hold the node), explain why that's true, and close with the cost/trade-off so the answer sounds balanced and mature.

Step 1 — What a linked list is (foundation)

A linked list stores elements in separate nodes scattered in memory; each node holds a value plus one or more pointers/references to other nodes. Unlike an array, the elements are not contiguous, so you can't jump to index k in O(1) — you must follow pointers from a known end. You keep a head reference (and often a tail) to enter the list.

Step 2 — Singly vs. doubly linked

Singly linked list (SLL): each node has { value, next }. You can only move forward (head → tail). To find the node before a given node, you must walk from the head again.

Doubly linked list (DLL): each node has { value, prev, next }. It stores two pointers, so from any node you can move both forward and backward. The first node's prev is null (or points to a sentinel), and the last node's next is null.

CODE
SLL:  head -> [A|·] -> [B|·] -> [C|null]

DLL:  null <- [·|A|·] <-> [·|B|·] <-> [·|C|·] -> null
             (prev|val|next)
Step 3 — The advantage (pick one, explain it fully)

Headline advantage: O(1) deletion (and insertion) when you already hold a reference to the node — because you have its prev pointer.

Why this matters: to delete a node from an SLL, you must know its predecessor so you can rewire predecessor.next = node.next. But an SLL node has no back pointer, so you must scan from the head to find that predecessor — O(n). In a DLL, the node itself carries prev, so deletion is just:

CODE
node.prev.next = node.next
node.next.prev = node.prev

That's a constant number of pointer updates — O(1) — no traversal needed. The same holds for inserting before a given node.

A second, related advantage is bidirectional traversal: you can walk backward (e.g., an "undo" history, a browser back/forward stack, or an LRU cache that must move a recently-used node to the front and evict from the tail). SLLs can't do this without reversing the list or keeping extra bookkeeping. If asked for only one advantage, I lead with the O(1)-node-deletion point because it's the most cited and most consequential in real systems (it's exactly why a hash map + DLL powers an LRU cache).

Step 4 — The trade-off (shows balance)

A DLL isn't free:

Extra memory: every node stores an additional prev pointer (one extra word per node).
More pointer maintenance: every insertion/deletion must correctly update two links instead of one, so there are more chances to introduce bugs (dangling or crossed pointers), especially at the head/tail boundaries. Sentinel/dummy head and tail nodes are commonly used to remove those special cases.

So: use a DLL when you need backward traversal or frequent O(1) removal of arbitrary nodes; use an SLL when memory is tight and you only ever move forward (e.g., a simple stack or a hash-table bucket chain).

Summary / what a strong answer emphasizes

Define the node as {prev, value, next}, contrast the single vs. double pointer clearly, and give one advantage explained with the reason: because a DLL node knows its predecessor, deleting a known node is O(1), whereas an SLL needs an O(n) scan to find that predecessor. Mention bidirectional traversal and the real-world LRU-cache use case, then acknowledge the extra memory and pointer-management cost. Common mistakes: saying "DLL is faster" without saying at what and why; forgetting to update prev on the neighbor during deletion; or ignoring null checks at the head/tail (which crash on the boundary nodes).

**Comparison:**

```
Aspect	Singly linked list	Doubly linked list
Pointers per node	1 (next)	2 (prev, next)
Traversal	Forward only	Forward and backward
Delete a known node	O(n) — must scan for predecessor	O(1) — prev is on the node
Memory per node	Lower	Higher (one extra pointer)
Pointer bookkeeping	Simpler	More links to maintain
Typical uses	Stack, hash-bucket chains	LRU cache, undo/redo, deque
```

**Code:**

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.prev = None   # <-- the extra pointer vs. a singly linked list
        self.next = None

class DoublyLinkedList:
    def __init__(self):
        self.head = None
        self.tail = None

    def append(self, value):
        node = Node(value)
        if not self.head:                 # empty list
            self.head = self.tail = node
        else:
            node.prev = self.tail
            self.tail.next = node
            self.tail = node
        return node

    def delete(self, node):
        """O(1): we have node.prev directly, no head-to-node scan needed."""
        if node.prev:
            node.prev.next = node.next
        else:                              # node was the head
            self.head = node.next
        if node.next:
            node.next.prev = node.prev
        else:                              # node was the tail
            self.tail = node.prev
        node.prev = node.next = None

    def traverse_backward(self):
        vals, cur = [], self.tail
        while cur:                         # not possible in a singly linked list
            vals.append(cur.value)
            cur = cur.prev
        return vals
```

**Example input / output:**

```text
dll = DoublyLinkedList()
a = dll.append("A"); b = dll.append("B"); c = dll.append("C")

dll.traverse_backward()   -> ["C", "B", "A"]   # backward walk (SLL cannot do this)

dll.delete(b)             # O(1): rewires A<->C using b.prev directly
# list is now A <-> C
dll.traverse_backward()   -> ["C", "A"]
```

**Flow:**

1. Doubly linked list after append A, B, C:
2. null <-- [prev|A|next] <--> [prev|B|next] <--> [prev|C|next] --> null
3. ^head                                     ^tail
4. Delete B (O(1)): set A.next = C and C.prev = A
5. null <-- [prev|A|next] <--> [prev|C|next] --> null

