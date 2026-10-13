---
type: "practice"
difficulty: "Basic"
---

# What is the time complexity of two nested loops both running n times?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Nested Loop Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Nested_Loop_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The question is: "What is the time complexity of two nested loops both running n times?" At face value this looks trivial, but in an interview the interviewer is probing whether you truly understand how to count the work an algorithm does as a function of input size, and whether you can reason about nesting rather than just recite "O(n²)". A strong candidate states the answer, derives it from first principles, notes the standard exceptions (loops that don't both run the full n times), and connects it to real code.

#### Assumptions
/ clarifying questions I'd state out loud:

Both loops iterate exactly n times, and the inner loop's range does not depend on the outer loop's variable (i.e. it's a fixed 0..n, not 0..i).
The body of the inner loop does a constant amount of work — O(1) — not another loop or a call that itself costs O(n).
We're talking about time complexity (number of basic operations), asymptotically, ignoring constant factors.

Plan: Count how many times the innermost statement runs → express as a function of n → drop constants/lower-order terms → state Big-O. Then cover the common variations so the answer is complete.

First principles — how to count the work

Big-O measures how the number of basic operations grows as the input size n grows. The rule for loops is simple:

A single loop that runs n times and does O(1) work per iteration is O(n) — it performs roughly n operations.
Nesting multiplies. For every one iteration of the outer loop, the inner loop runs its full course. So the inner body executes (outer iterations) × (inner iterations) times.

For our case both loops run n times:

CODE
outer runs n times
  for each outer iteration, inner runs n times
    inner body = O(1)

Total inner-body executions = n × n = n². Dropping the constant per-operation cost, the time complexity is O(n²) — "quadratic" time.

Why we drop the constants

If the inner body does, say, 3 operations, the true count is 3n². Big-O ignores constant multipliers (3) and lower-order terms because they don't change the growth rate — what dominates as n gets large is the n² term. So 3n² + 5n + 7 is still O(n²). This is why "two nested loops of size n" collapses to O(n²) regardless of the exact constant work inside.

Concrete intuition with numbers

Think of a multiplication table or a grid of size n×n:

n	Inner-body executions (n²)
10	100
100	10,000
1,000	1,000,000
10,000	100,000,000

Notice: when n grows 10×, the work grows 100×. That squared growth is the signature of O(n²) and is exactly why quadratic algorithms become slow for large inputs (e.g. a naive O(n²) pairwise comparison over 100,000 items is 10 billion operations — usually too slow).

Important variations (this is what makes the answer complete)

The clean answer is O(n²), but a great candidate flags when it isn't:

Inner loop depends on the outer index (e.g. for j in range(i, n)): the inner runs n, n−1, n−2, … 1 times. The total is n + (n−1) + … + 1 = n(n+1)/2, which is still O(n²) — the constant ½ drops out. So a triangular nested loop is also quadratic, just with half the operations.
Inner loop runs a constant number of times (e.g. always 5): total is 5n → O(n), not O(n²). "Nested" alone doesn't guarantee quadratic — both loops must scale with n.
Inner loop runs a different variable m (e.g. iterating an n×m matrix): complexity is O(n·m), which is only O(n²) when m = n.
Inner body itself is O(n) (e.g. a substring copy or another hidden loop): then it's n × n × n = O(n³). Always check the body, not just the loop headers.
Space complexity note

If the two loops only use a few scalar variables (counters, an accumulator), the space complexity is O(1) — nesting affects time, not space, unless you're allocating memory proportional to the iterations (e.g. building an n×n result matrix, which would be O(n²) space).

Summary — what a strong answer emphasizes

Two nested loops that each run n times with O(1) work inside give O(n²) time, O(1) space, because the inner statement executes n × n = n² times and constants are dropped. The key insight is that nesting multiplies the iteration counts. The candidate should also caveat: this holds only when both loops truly scale with n and the inner range is independent of the outer index; a triangular loop is still O(n²), a fixed inner loop is O(n), and different bounds give O(n·m).

#### Common mistakes
saying "nested loop = O(n²)" without checking whether both loops actually run n times; forgetting that n(n+1)/2 simplifies to O(n²); missing a hidden O(n) operation inside the body that would push it to O(n³).

Likely follow-up probes: "What if the inner loop starts at i?" (still O(n²)), "What if the inner body does a .contains() on a list?" (could become O(n³)), and "How would you reduce an O(n²) pairwise-comparison to O(n) or O(n log n)?" (hashing / sorting).

**Comparison:**

```
Loop structure	Inner-body executions	Big-O
both loops run n, inner independent of i	n × n = n²	O(n²)
inner runs from i to n (triangular)	n(n+1)/2	O(n²)
inner runs a fixed constant (e.g. 5)	5n	O(n)
outer n, inner m (different sizes)	n × m	O(n·m)
inner body itself is O(n)	n × n × n	O(n³)
```

**Code:**

```python
# Case A: classic two nested loops, both run n times -> O(n^2)
def count_pairs(n):
    ops = 0
    for i in range(n):        # outer: n iterations
        for j in range(n):    # inner: n iterations, independent of i
            ops += 1          # O(1) body
    return ops                # ops == n * n == n^2

# Case B: triangular nesting -> still O(n^2)
def triangular(n):
    ops = 0
    for i in range(n):
        for j in range(i, n): # runs n, n-1, ... 1  => n(n+1)/2
            ops += 1
    return ops                # ops == n*(n+1)//2, which is O(n^2)

# Case C: inner loop is constant -> O(n), NOT O(n^2)
def linear(n):
    ops = 0
    for i in range(n):
        for _ in range(5):    # fixed 5, does not scale with n
            ops += 1
    return ops                # ops == 5n -> O(n)
```

**Example input / output:**

```

Dry run of Case A count_pairs(n):

CODE
Copy
count_pairs(1)  -> ops = 1     (1 x 1)
count_pairs(2)  -> ops = 4     (2 x 2)
count_pairs(3)  -> ops = 9     (3 x 3)
count_pairs(10) -> ops = 100   (10 x 10)

Doubling n from 5 -> 10 changes ops from 25 -> 100 (4x), confirming quadratic growth.

Dry run of Case B triangular(n):

CODE
Copy
triangular(3) -> 3 + 2 + 1 = 6   = 3*4/2   (still O(n^2))
triangular(4) -> 4 + 3 + 2 + 1 = 10 = 4*5/2
```

