---
type: "practice"
difficulty: "Basic"
---

# What is a list comprehension? Rewrite a for-loop that builds a list of squares as a comprehension.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Membership and Collection Complexity](<../../../../TOOLS/Python/Python_Language/Membership_and_Collection_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Two things. (1) Define what a list comprehension is in Python. (2) Take an ordinary for-loop that builds a list of squares and rewrite it as a comprehension. This is a basic-concept, fresher-level question.

What the interviewer is probing: Do you know the idiomatic, "Pythonic" way to build lists? Can you read and transform imperative loop code into declarative comprehension syntax? A strong candidate also mentions readability, the fact that a comprehension is an expression that produces a new list, and knows when not to use one.

#### Assumptions
"Squares" means the squares of a range of numbers (e.g., 0..9). I'll keep the example concrete and note the general shape.

Plan: Define the concept from first principles → give the loop → rewrite it → generalize the syntax (with if filter and nested loops) → mention siblings (dict/set/generator comprehensions) → summarize and warn about common mistakes.

What a list comprehension is

A list comprehension is a compact, single-expression way to create a new list by transforming and/or filtering an existing iterable (like a range, list, string, or generator). Instead of writing an empty list and appending inside a loop, you describe what the list should contain in one line. It is an expression, so it evaluates to a list value that you can assign, return, or pass directly to a function.

Conceptually it mirrors math set-builder notation: "the set of x² for every x in 0..9" becomes [x*x for x in range(10)].

The for-loop version (building squares)
PYTHON
squares = []
for x in range(10):
    squares.append(x * x)
# squares == [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

Here we (1) create an empty list, (2) loop, (3) compute x*x, (4) append each result. Four concerns spread across four lines, with a mutable accumulator (squares) that lives outside the loop.

The comprehension version
PYTHON
squares = [x * x for x in range(10)]
# squares == [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

Same result, one line, no explicit append, no pre-declared empty list. This is the idiomatic Python way.

The general syntax

The full shape of a list comprehension is:

CODE
[ <expression> for <item> in <iterable> if <condition> ]
<expression> — what to put in the list for each item (here x * x).
for <item> in <iterable> — where the items come from.
if <condition> — an optional filter; only items passing it are included.

With a filter — squares of only the even numbers:

PYTHON
even_squares = [x * x for x in range(10) if x % 2 == 0]
# [0, 4, 16, 36, 64]

With a conditional expression (transform differently, but keep every item — note the if...else goes before the for):

PYTHON
labels = ["even" if x % 2 == 0 else "odd" for x in range(5)]
# ['even', 'odd', 'even', 'odd', 'even']

Nested loops — flatten a matrix (loops read left-to-right, outer first):

PYTHON
matrix = [[1, 2], [3, 4]]
flat = [n for row in matrix for n in row]
# [1, 2, 3, 4]
Related comprehensions

The same syntax produces other containers:

Set comprehension: {x * x for x in range(10)} → a set of unique squares.
Dict comprehension: {x: x * x for x in range(5)} → {0:0, 1:1, 2:4, 3:9, 4:16}.
Generator expression: (x * x for x in range(10)) → a lazy generator (parentheses), which computes values one at a time and uses O(1) memory instead of building the whole list. Use this for large data you only iterate once, e.g. sum(x*x for x in range(10**6)).
Why comprehensions are preferred
Readability: intent ("a list of squares") is right there in one expression.
Slightly faster: the append happens in optimized C internally rather than a Python-level .append call each iteration.
No leaky accumulator: you don't manage a mutable list by hand, and (in Python 3) the loop variable x does not leak into the surrounding scope.

#### Common mistakes
/ follow-up probes
Overusing them. If the logic has side effects (printing, writing to a DB) or needs multiple statements/try-except, use a regular loop — a comprehension should build a value, not do work.
Cramming too much. A deeply nested comprehension with several filters is harder to read than a loop. Favor clarity.
Confusing filter placement. if at the end filters items; if...else at the front is a ternary that transforms every item. Interviewers love this distinction.
Building a huge list when you only iterate once. Prefer a generator expression to save memory.
[...] vs (...): brackets build a full list eagerly; parentheses build a lazy generator.
Summary — what a strong answer emphasizes

A list comprehension is a one-line expression [expr for item in iterable if cond] that builds a new list by mapping/filtering an iterable. The squares rewrite is simply squares = [x*x for x in range(10)], replacing the empty-list-plus-append loop. A standout candidate also notes the optional filter, the front-of-clause if/else ternary, nested loops, the dict/set/generator siblings, the readability and small speed win, and the judgment to fall back to an ordinary loop when there are side effects or the expression gets unreadable.

**Comparison:**

```
Aspect	for-loop + append	List comprehension
Lines	3–4	1
Style	Imperative (how)	Declarative (what)
Accumulator	Manual empty list	Built for you
Speed	Slower (Python-level append)	Slightly faster (C-level)
Best for	Side effects / multi-statement logic	Building a value from an iterable
```

**Code:**

```python
# For-loop version
squares = []
for x in range(10):
    squares.append(x * x)

# List comprehension version (idiomatic)
squares = [x * x for x in range(10)]

# With a filter: squares of even numbers only
even_squares = [x * x for x in range(10) if x % 2 == 0]

# Conditional expression (ternary before the for)
labels = ["even" if x % 2 == 0 else "odd" for x in range(5)]

# Nested loops: flatten a matrix
flat = [n for row in [[1, 2], [3, 4]] for n in row]

# Siblings
sq_set  = {x * x for x in range(10)}        # set comprehension
sq_dict = {x: x * x for x in range(5)}      # dict comprehension
sq_gen  = (x * x for x in range(10))        # generator expression (lazy)
```

**Example input / output:**

```text
[x * x for x in range(10)]
-> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

[x * x for x in range(10) if x % 2 == 0]
-> [0, 4, 16, 36, 64]

["even" if x % 2 == 0 else "odd" for x in range(5)]
-> ['even', 'odd', 'even', 'odd', 'even']

[n for row in [[1, 2], [3, 4]] for n in row]
-> [1, 2, 3, 4]

{x: x * x for x in range(5)}
-> {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

