---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a queue using two stacks?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queues and Stacks](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queues_and_Stacks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what's really being asked

I'm asked to implement a FIFO queue (enqueue, dequeue, and usually peek/empty) using only two LIFO stacks as the underlying storage — no arrays, no linked lists directly, just push/pop/top/isEmpty on the two stacks. The interviewer is probing whether I understand that a stack reverses order, and that reversing twice restores the original order — which is the whole trick to turn LIFO into FIFO. They also want to see whether I reach for the naive per-operation approach or the amortized O(1) approach, and whether I can prove the amortized bound.

Clarifying questions:

Do we need to optimize enqueue or dequeue? There are two designs — one makes enqueue O(1), the other makes dequeue O(1). I'll present the amortized-O(1) lazy design (fast enqueue, amortized-fast dequeue), which is the one interviewers expect.
Is it single-threaded? I'll assume yes; I'll mention thread-safety as a follow-up.
Should dequeue/peek on an empty queue throw or return a sentinel? I'll throw (or return None), consistent with typical queue contracts.

Plan: explain why one stack can't do it, give the naive "move everything on every dequeue" version, then the lazy two-stack version, prove its amortized cost, dry-run it, and cover edge cases.

Why a stack alone doesn't work

A stack is LIFO: the last element pushed is the first popped. A queue is FIFO: the first element enqueued must be the first dequeued. If I push 1, 2, 3 onto a stack and pop, I get 3, 2, 1 — the reverse of what a queue needs. The core idea: pouring a stack into another stack reverses it. So if I pour once, the bottom element (oldest) rises to the top, ready to be popped in FIFO order.

The two designs

I'll use two stacks: in (receives new items) and out (serves items).

Design A — costly enqueue (O(n) enqueue, O(1) dequeue): on every enqueue, move everything from out to in, push the new item, move it all back. This keeps out always in FIFO order so dequeue is O(1). It works but each enqueue is O(n) — usually not what you want.

Design B — lazy / amortized (O(1) enqueue, amortized O(1) dequeue) — the standard answer:

enqueue(x): just in.push(x). Always O(1).
dequeue(): if out is empty, pour all of in into out (this reverses in, so the oldest element ends up on top of out). Then out.pop(). If out is not empty, just out.pop().
peek(): same as dequeue but out.top() instead of pop.
empty(): both stacks empty.

The crucial correctness point: I only refill out when it is empty. This preserves FIFO order globally, because everything currently in out is older than everything in in, and I never interleave a pour with a partially-full out.

Amortized analysis (why it's O(1) even though a pour is O(n))

Each element is pushed to in once, popped from in once, pushed to out once, and popped from out once — exactly 4 stack operations over its entire lifetime, no matter how the operations interleave. So m total queue operations do O(m) stack work → O(1) amortized per operation.

A cleaner way to say it (accounting method): charge each enqueue 3 units — 1 for the immediate push, and 2 saved as "credit" to pay for the future transfer (pop from in, push to out) and eventual pop from out. Every dequeue is then paid for by pre-saved credit, so its amortized cost is O(1). The worst-case single dequeue is still O(n) (when it triggers a full pour), which matters for real-time systems — I'd flag that trade-off.

Space: O(n) total across both stacks.
enqueue: O(1) worst case.
dequeue/peek: O(1) amortized, O(n) worst case.
Edge cases & pitfalls
Dequeue/peek on empty queue: check empty() first; throw or return sentinel. Don't pour an empty in and then pop an empty out.
Don't refill out while it's non-empty — that's the classic bug; it scrambles FIFO order.
Peek must not lose the element — use top(), not pop(), or push it back.
Interleaving enqueue/dequeue is handled naturally because refills are lazy.

#### Follow-ups
I'd anticipate
Which is better, two-stacks-queue or two-queues-stack? They're duals; the queue-from-stacks is the more common ask.
Real-time (worst-case O(1)) queue from stacks: achievable with a more complex incremental-transfer scheme (Okasaki-style), moving one element per operation instead of a full batch — I'd mention it exists but note the amortized version is what's expected here.
Thread safety: wrap with a lock, or use two lock-free stacks with care; the lazy transfer complicates concurrency, so a single mutex around the pair is the pragmatic choice.
What a strong answer emphasizes

State the two-stack in/out design, the "reverse-twice = original order" insight, only refill when out is empty, and prove amortized O(1) via the 4-ops-per-element argument while acknowledging O(n) worst-case dequeue. See code_blocks for a clean implementation and input_output for a dry run.

**Comparison:**

```
Operation	Design A (fast dequeue)	Design B (lazy, standard)
enqueue	O(n)	O(1) worst case
dequeue	O(1)	O(1) amortized, O(n) worst
peek	O(1)	O(1) amortized
space	O(n)	O(n)
when to use	dequeue-heavy, rare enqueue	general purpose (default)
```

**Code:**

```

class QueueUsingStacks: def init(self): self._in = [] # receives enqueues self._out = [] # serves dequeues (reversed order)

CODE
Copy
def enqueue(self, x):
    self._in.append(x)            # O(1)

def _transfer_if_needed(self):
    # Only refill when out is empty -> preserves FIFO order
    if not self._out:
        while self._in:
            self._out.append(self._in.pop())

def dequeue(self):
    self._transfer_if_needed()
    if not self._out:
        raise IndexError("dequeue from empty queue")
    return self._out.pop()         # amortized O(1)

def peek(self):
    self._transfer_if_needed()
    if not self._out:
        raise IndexError("peek from empty queue")
    return self._out[-1]

def empty(self):
    return not self._in and not self._out

def __len__(self):
    return len(self._in) + len(self._out)
CODE
Copy

```java
// Java equivalent (amortized O(1))
import java.util.*;
class MyQueue {
    private Deque<Integer> in  = new ArrayDeque<>();
    private Deque<Integer> out = new ArrayDeque<>();

    public void push(int x) { in.push(x); }           // enqueue, O(1)

    private void transfer() {
        if (out.isEmpty())
            while (!in.isEmpty()) out.push(in.pop());
    }
    public int pop()  { transfer(); return out.pop(); }   // amortized O(1)
    public int peek() { transfer(); return out.peek(); }
    public boolean empty() { return in.isEmpty() && out.isEmpty(); }
}
```

**Example input / output:**

```text
Operations and state (in = enqueue stack, out = serve stack; top of each shown on the right):

| Op            | in (top->)    | out (top->) | returns | note |
|---------------|---------------|-------------|---------|------|
| enqueue(1)    | [1]           | []          |         | |
| enqueue(2)    | [1,2]         | []          |         | |
| enqueue(3)    | [1,2,3]       | []          |         | |
| dequeue()     | []            | [3,2] (top=1)| 1      | out empty -> pour, reverses to 1,2,3; pop 1 |
| enqueue(4)    | [4]           | [3,2]       |         | new item goes to in |
| dequeue()     | [4]           | [3]         | 2       | out non-empty -> just pop, NO refill |
| dequeue()     | [4]           | []          | 3       | pop 3, out now empty |
| dequeue()     | []            | [4]         | 4       | out empty -> pour 4; pop 4 |
| dequeue()     | []            | []          | ERROR   | empty queue |

Result order returned: 1, 2, 3, 4  -> correct FIFO.
```

**Flow:**

1. dequeue
2. yes
3. no
4. yes
5. no
6. dequeue
7. out empty?
8. pour ALL of in into outreverses order
9. skip pour
10. out still empty?
11. out.pop -> return
12. throw: empty queue
13. enqueue
14. enqueue x
15. in.push x

