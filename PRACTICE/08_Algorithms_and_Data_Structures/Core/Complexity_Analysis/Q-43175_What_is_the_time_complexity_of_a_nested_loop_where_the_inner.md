---
type: "practice"
difficulty: "Basic"
---

# What is the time complexity of a nested loop where the inner loop runs a fixed constant number of times?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Nested Loop Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Nested_Loop_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question asks: You have two nested loops. The outer loop runs n times, but the inner loop runs a fixed constant number of times (say 5, or 100 — some number that does NOT depend on n). What is the time complexity? The answer is O(n) (linear), not O(n²).

What the interviewer is probing (basic-concept / fresher level): Do I understand that Big-O ignores constant factors? A loop that always runs the same fixed number of times — no matter how big n is — is just a constant amount of work bolted onto each outer step. Nested does not automatically mean quadratic. I need to show I can tell "depends on n" apart from "fixed constant."

Assumption: The inner loop body is O(1) and the inner loop's bound is a literal constant (like for j in range(5)), not something like for j in range(n) or for j in range(i). I'll say so explicitly, because that distinction is the whole point.

Plan: Count total iterations → show it's constant × n → apply the drop-constants rule → arrive at O(n) → contrast with the case where the inner bound does depend on n.

First principles: count the total work

The total number of times the innermost body runs = (outer iterations) × (inner iterations per outer pass).

CODE
for i = 0 .. n-1:        # outer runs n times
    for j = 0 .. 4:      # inner runs 5 times, FIXED, independent of n
        <O(1) work>
Outer runs n times.
Inner runs 5 times every pass — 5 whether n is 10 or 10 million.
Total body executions = n × 5 = 5n.
Applying the Big-O rule: drop constant factors

Big-O measures how runtime grows as n grows, and it deliberately ignores constant multipliers. 5n, 100n, 2n all grow linearly — double n and the work doubles. So:

5n → O(n).

Time complexity = O(n) (linear). The fixed inner loop is just a constant factor. It makes the program a bit slower by a constant amount per step, but it does not change the growth curve. That is the core idea: a nested loop is only O(n²) when the inner loop's count grows with n. If the inner count is a constant, the two loops together are still linear.

Why not O(n²)?

O(n²) happens when the inner loop also runs roughly n times, e.g. for j in range(n) — then total = n × n = n². Here the inner runs a fixed 5, so total = n × 5. The presence of two nested for loops is NOT what makes something quadratic; what the loops count up to is. Fixed constant inside → linear.

Think of it concretely: if the inner loop always does 5 things, then processing each of n items costs a fixed "5 units." That's the same shape as doing 1 unit per item (O(n)) — just 5× the constant, which Big-O hides.

Space complexity

The loops use only a couple of counter variables and (fixed) constant inner iterations; nothing scales with n. So auxiliary space = O(1) (constant).

Numeric sanity check

For inner constant = 5:

n = 100 → 500 body executions.
n = 1,000 → 5,000.
n = 1,000,000 → 5,000,000.

Every time n grows 10×, the work grows 10× — a straight line, i.e. linear O(n). If it were O(n²), a 10× bigger n would mean 100× the work.

#### Common mistakes
(what freshers get wrong here)
"Two nested loops = O(n²)." Wrong. Only when both bounds depend on n. A fixed inner bound keeps it O(n).
Keeping the constant: "O(5n)." Big-O drops constant factors — write O(n), not O(5n).
Confusing a constant with i or n. for j in range(5) is constant → O(n). for j in range(n) → O(n²). for j in range(i) → depends on i → O(n²) (triangular sum). Read the inner bound carefully.
Forgetting the body cost. If the fixed inner loop's body did O(n) work (e.g., scanned the whole array), you'd multiply: n × 5 × n = O(n²). Here the body is O(1), so it stays O(n).
Summary — what a strong basic answer says

Total iterations = n × constant = constant·n, and Big-O drops the constant, so the time complexity is O(n) (linear) with O(1) space. The one takeaway: a nested loop is quadratic only if the inner loop's count grows with n; a fixed-constant inner loop is just a constant factor, so the whole thing stays linear.

**Comparison:**

```
Inner loop bound	Inner runs	Total iterations	Time complexity
range(5) (constant)	5 (fixed)	5·n	O(n)
range(100) (constant)	100 (fixed)	100·n	O(n)
range(n) (depends on n)	n	n·n	O(n²)
range(i) (depends on outer)	~n/2 avg	n(n-1)/2	O(n²)
Metric	Result
Total body executions	constant × n
Time complexity	O(n) (linear)
Auxiliary space	O(1)
Why not O(n²)	inner count does not grow with n
```

**Code:**

```python
# Inner loop runs a FIXED constant number of times (5), independent of n.
def fixed_inner(n):
    count = 0
    for i in range(n):        # outer: n times
        for j in range(5):    # inner: 5 times, constant (not n, not i)
            count += 1        # O(1) body
    return count              # == 5 * n   ->   O(n)

# Contrast: inner depends on n -> quadratic.
def variable_inner(n):
    count = 0
    for i in range(n):        # outer: n times
        for j in range(n):    # inner: n times  ->  n * n
            count += 1
    return count              # == n * n   ->   O(n^2)
```

**Example input / output:**

```

Dry run of fixed_inner for n = 4 (inner always runs 5):

CODE
Copy
i = 0 -> inner runs 5 times  (running total 5)
i = 1 -> inner runs 5 times  (running total 10)
i = 2 -> inner runs 5 times  (running total 15)
i = 3 -> inner runs 5 times  (running total 20)
total = 4 * 5 = 20 = 5n   ->   O(n)

Growth check (fixed inner = 5):

CODE
Copy
 n         | total = 5n   | O(n^2) would be
 100       | 500          | 10,000
 1,000     | 5,000        | 1,000,000
 1,000,000 | 5,000,000    | 1,000,000,000,000
n grows 10x  ->  work grows 10x  ->  LINEAR (not quadratic).
```

**Flow:**

1. No — fixed constant, e.g.range(5)
2. Yes — e.g. range(n)
3. Nested loop
4. Does inner loop count dependon n?
5. total = constant × n
6. drop constant → O(n) linear
7. total = n × n → O(n²)

