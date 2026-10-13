---
type: "practice"
difficulty: "Basic"
---

# What is a stack, and give two real-world programming scenarios where you'd use one.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queues and Stacks](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queues_and_Stacks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

Two parts: (1) define a stack precisely, and (2) give two concrete programming scenarios where a stack is the natural fit. The interviewer is checking that I know the LIFO abstract data type, its core operations and their cost, and — more importantly — that I can map it to real problems rather than just reciting a definition. A weak answer stops at "last in, first out." A strong answer names the operations, states their O(1) cost, and ties each scenario back to why LIFO is exactly what the problem needs.

#### Assumptions
/ clarifications: I'll treat "stack" as the classic abstract data type (not the hardware call stack specifically, though I'll mention it as scenario 1). No ordering guarantees beyond LIFO; I'll note it can be backed by an array or a linked list.

Definition — from first principles

A stack is a linear collection with a single point of access — the top — and it follows LIFO: Last In, First Out. The most recently added item is the first one removed. Picture a stack of plates: you add and remove only from the top; you cannot pull one out of the middle without disturbing the ones above it.

Core operations, all O(1):

push(x) — add x on top.
pop() — remove and return the top element.
peek() / top() — read the top without removing it.
isEmpty() — whether the stack has any elements.
(often) size() — number of elements.

#### Implementation
either a dynamic array (push/pop at the end — amortized O(1)) or a singly linked list (push/pop at the head — O(1)). The array version has better cache locality; the linked version never needs to resize. Trying to pop/peek an empty stack is an error (stack underflow); a fixed-size array stack can also overflow.

The essence: a stack deliberately restricts access to one end so it can guarantee O(1) operations and model any process that must be unwound in reverse order of how it was built up.

Scenario 1 — Function call management (the call stack)

Every running program uses a stack to manage function calls. When function A calls B, and B calls C, the runtime pushes a stack frame (return address, parameters, local variables) for each call. When C returns, its frame is popped, and control returns to B; then B returns and its frame pops, back to A. This is inherently LIFO: the most recently called function is the first to finish and return.

Why a stack is the right structure: calls nest perfectly — you always finish the innermost (most recent) call before resuming the caller. This is also exactly how recursion works, and why deep or infinite recursion produces a StackOverflowError — you push frames faster than you pop them until the memory reserved for the stack is exhausted. Converting a recursive algorithm to an iterative one usually means replacing the implicit call stack with an explicit stack that you manage yourself (e.g., iterative DFS or iterative tree traversal).

Scenario 2 — Matching / balancing symbols (expression & syntax validation)

Checking whether brackets are balanced — ([{}]) — is a textbook stack problem, and it generalizes to compilers/parsers, JSON/XML validators, and editor "jump to matching brace" features. Scan left to right: push every opening bracket; on a closing bracket, pop and check it matches the expected opening. If it doesn't match, or the stack is empty when you need to pop, or the stack isn't empty at the end, the string is unbalanced.

Why a stack fits: the most recently opened bracket must be the first to close — LIFO exactly. The same mechanism drives undo/redo in editors (push each action; undo pops the latest), browser back-button history (push each page; back pops the most recent), evaluating arithmetic expressions (converting infix to postfix / evaluating postfix), and DFS graph traversal (an explicit stack of nodes to visit). All share the trait: process things in the reverse order they arrived.

Summary — what a strong answer emphasizes
Nail the definition: LIFO, single access point (top), O(1) push/pop/peek.
Mention it can be array- or linked-list-backed, and the underflow/overflow edge cases.
Give two scenarios and, crucially, explain why LIFO is the right model for each: function call/recursion management (innermost call finishes first) and balanced-symbol/undo/back-history/DFS (most recent open closes first).
Contrast with a queue (FIFO) if asked — that's the natural follow-up: use a stack when order must be reversed, a queue when order must be preserved.

#### Common mistakes
reciting only "LIFO" with no operations or cost; giving two scenarios that are secretly the same one; forgetting the empty-stack error case; confusing stack (LIFO) with queue (FIFO).

**Comparison:**

```
Operation	Meaning	Time
push(x)	add x to the top	O(1)
pop()	remove & return top	O(1)
peek()/top()	read top, no removal	O(1)
isEmpty()	any elements?	O(1)
	Stack	Queue
Order	LIFO (last in, first out)	FIFO (first in, first out)
Access ends	one end (top)	two ends (rear in, front out)
Use when	order must be reversed / unwound	order must be preserved
```

**Code:**

```
Balanced-brackets validator — the archetypal stack use case.

def is_balanced(s): pairs = {')': '(', ']': '[', '}': '{'} stack = [] for ch in s: if ch in '([{': stack.append(ch) # push opening elif ch in ')]}': if not stack or stack.pop() != pairs[ch]: return False # nothing to match, or mismatch return not stack # leftover openings => unbalanced

CODE
Copy

```python
# A minimal stack (array-backed); all ops O(1).
class Stack:
    def __init__(self):      self._d = []
    def push(self, x):       self._d.append(x)
    def pop(self):           return self._d.pop()      # raises if empty
    def peek(self):          return self._d[-1]
    def is_empty(self):      return not self._d
    def size(self):          return len(self._d)
```

**Example input / output:**

```text
is_balanced("([{}])")  -> True
is_balanced("([)]")    -> False   # ')' pops '[' which does not match
is_balanced("(((")     -> False   # stack non-empty at end
is_balanced("")        -> True

Call stack trace for A()->B()->C():
  push A | push B | push C | (C returns) pop C | pop B | pop A  -> empty
```

**Flow:**

1. goes on top
2. removes 3 first
3. Stack [LIFO]
4. top -> 3
5. 2
6. 1 -> bottom
7. push 4
8. pop

