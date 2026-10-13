---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a stack that returns the minimum element in O(1)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ O(1) Randomized Set](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/O(1)_Randomized_Set.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what's really being asked

I need to design a stack that supports push, pop, top, and an extra getMin() that returns the current minimum element, with every operation running in O(1). The naive getMin — scan the whole stack — is O(n), so the interviewer is really probing: can I trade a little space to make min a constant-time query, and do I correctly maintain the min as elements are pushed and popped? The subtlety is that pop can remove the current minimum, so I must be able to recover the previous minimum instantly.

Clarifying questions:

Duplicates of the minimum? e.g. push 2, 2, pop once — the min must still be 2. My design must handle equal-to-min values (I'll push on <=, not <).
Behavior on empty stack for pop/top/getMin? I'll throw / return a sentinel.
Space budget? The clean solution uses O(n) auxiliary space (an aux stack). There's also an O(1)-extra-space encoding trick if they push back; I'll present the aux-stack version as primary and mention the trick.

Plan: show why scanning is too slow, give the two-stack solution, prove O(1) for all ops, handle the duplicate-min edge case, dry-run it, then present the space-optimized "pairs" and "encoding" alternatives.

From first principles

The obstacle: a plain stack only gives you the top. The minimum could be buried anywhere, and when you pop you might remove it — so you need the history of minimums, not just the current one.

Key insight — remember the min "so far" alongside each element. At the moment I push a value, I know the min of everything currently in the stack. If I store that min next to the element (or in a parallel stack), then after any pop the min of the remaining stack is simply "the min recorded by the new top." That makes getMin a single lookup — O(1).

Primary solution — two stacks

Keep the main stack s and an auxiliary min-stack minS. minS is kept the same height as s; minS.top() is always the minimum of everything currently in s.

push(x): s.push(x). If minS is empty, push x; else push min(x, minS.top()).
pop(): pop from both s and minS. (minS shrinks in lockstep, so its new top is the min of the remaining elements.)
top(): s.top().
getMin(): minS.top().

Every operation is a constant number of stack primitives → O(1) each. Space: O(n) extra for minS.

Duplicate-min correctness: because on each push I store min(x, previous_min), pushing 2 then 2 records 2, 2 in minS; popping one still leaves 2 on top of minS. This is exactly why the parallel-stack (or <= variants below) beats a "single value" approach — a lone min variable can't recover the previous min after the current one is popped.

Space optimization 1 — store the min only when it changes

Instead of a full-height minS, push onto minS only when a new value is ≤ current min, and pop from minS only when the popped value equals minS.top(). This shrinks minS in the common case where the min rarely changes, though it's still O(n) worst case. The ≤ (not <) is essential so duplicate minimums are popped correctly.

Space optimization 2 — encoding trick (O(1) extra space)

Store a single min variable and encode into the main stack. When pushing x < min, push the encoded value 2*x − min (which is less than x, a flag) and update min = x. On pop, if the popped value < min (the flag), the real popped element was the old min, and you recover the previous min as 2*min − popped. This uses no auxiliary stack (O(1) extra space) but relies on integer arithmetic that can overflow — I'd use 64-bit / long for the encoded value and mention that caveat. In an interview I'd lead with the clean two-stack version and offer this as the "can you do it in O(1) space?" follow-up.

Complexity summary
push / pop / top / getMin: O(1) each (all variants).
Space: O(n) for two-stack designs; O(1) extra for the encoding trick.
Edge cases & pitfalls
getMin/top/pop on empty stack: guard and throw / return sentinel.
Duplicate minimums: must use <= when deciding to record/remove the min, or store min(x, prevMin) every push. Using < here is the classic bug.
Overflow in the encoding trick — use wide integers.
Keeping the two stacks in lockstep — always push/pop both (in the full-height version) or the invariant breaks.

#### Follow-ups
I'd anticipate
getMax too? Symmetric — a second aux stack storing running max.
Min at each point in a queue / sliding window? Different structure (monotonic deque), but the "keep a monotonic history" idea generalizes.
Thread safety? Wrap the pair under one lock so the two stacks stay consistent.
What a strong answer emphasizes

Lead with the two-stack design and the invariant "minS.top() == min of s," prove O(1) for all four operations, explicitly handle duplicate minimums with <=/min(), and offer the O(1)-extra-space encoding trick with its overflow caveat. See code_blocks and the input_output dry run.

**Comparison:**

```
Design	push	pop	top	getMin	extra space	duplicate-min safe?
Scan on getMin	O(1)	O(1)	O(1)	O(n)	O(1)	yes but slow
Two stacks (full height)	O(1)	O(1)	O(1)	O(1)	O(n)	yes (store min each push)
Aux stack on-change (<=)	O(1)	O(1)	O(1)	O(1)	O(n), often less	yes if <= used
Encoding trick	O(1)	O(1)	O(1)	O(1)	O(1)	yes; watch overflow
```

**Code:**

```

class MinStack: def init(self): self._s = [] # main stack self._min = [] # parallel min stack; _min[-1] = min of _s

CODE
Copy
def push(self, x):
    self._s.append(x)
    self._min.append(x if not self._min else min(x, self._min[-1]))

def pop(self):
    if not self._s:
        raise IndexError("pop from empty stack")
    self._min.pop()
    return self._s.pop()

def top(self):
    if not self._s:
        raise IndexError("top from empty stack")
    return self._s[-1]

def get_min(self):
    if not self._min:
        raise IndexError("getMin from empty stack")
    return self._min[-1]   # O(1)
CODE
Copy

```java
// O(1) extra space encoding variant (uses long to avoid overflow)
class MinStack {
    private Deque<Long> s = new ArrayDeque<>();
    private long min;
    public void push(int x) {
        if (s.isEmpty()) { s.push((long) x); min = x; }
        else if (x >= min) { s.push((long) x); }
        else { s.push(2L * x - min); min = x; }   // encoded flag < x
    }
    public void pop() {
        long v = s.pop();
        if (v < min) min = 2 * min - v;            // recover previous min
    }
    public int top() {
        long v = s.peek();
        return (int) (v < min ? min : v);
    }
    public int getMin() { return (int) min; }      // O(1)
}
```

**Example input / output:**

```text
Two-stack version. s = main stack, minS = min stack (top shown rightmost):

| Op         | s              | minS           | getMin() | note |
|------------|----------------|----------------|----------|------|
| push(5)    | [5]            | [5]            | 5        | |
| push(3)    | [5,3]          | [5,3]          | 3        | min(3,5)=3 |
| push(7)    | [5,3,7]        | [5,3,3]        | 3        | min(7,3)=3 |
| push(3)    | [5,3,7,3]      | [5,3,3,3]      | 3        | duplicate min stored |
| pop() ->3  | [5,3,7]        | [5,3,3]        | 3        | still 3 (duplicate handled) |
| pop() ->7  | [5,3]          | [5,3]          | 3        | |
| pop() ->3  | [5]            | [5]            | 5        | min correctly recovers to 5 |
| top()      | [5]            | [5]            | 5        | returns 5 |
| pop() ->5  | []             | []            |  -       | now empty |
| getMin()   | []             | []            | ERROR    | empty stack |

All of push/pop/top/getMin were single O(1) stack operations.
```

**Flow:**

1. yes
2. no
3. push x
4. minS empty?
5. minS.push x
6. minS.push min of x andminS.top
7. s.push x
8. pop
9. pop s AND pop minS inlockstep
10. getMin
11. return minS.top -- O1

