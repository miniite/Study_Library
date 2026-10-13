---
type: "practice"
difficulty: "Intermediate"
---

# Trace the complexity of a loop where j goes from 0 to i for each i from 0 to n. What's the total?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Nested Loop Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Nested_Loop_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question asks: For the double loop where j runs from 0 to i while i runs from 0 to n, what is the total number of inner-loop iterations, and therefore the time complexity? The answer is n(n+1)/2 ≈ n²/2 iterations → Θ(n²) — and the interviewer wants to see me derive it, not just quote it.

What the interviewer is probing (intermediate level): Do I know that a triangular loop is NOT simply n × n? Can I sum an arithmetic series to get the exact count n(n+1)/2, and then correctly collapse it to Θ(n²) by dropping the constant factor? Do I understand why halving the work (a triangle is half the square) does not change the complexity class?

Assumption / clarifying question: I'll assume the loop body is O(1). If it weren't, I'd multiply the iteration count by the body cost. I'd also confirm whether the bound is inclusive (j <= i, giving i+1 iterations) or exclusive (j < i, giving i) — it changes the exact count by a lower-order term but not the Θ class. I'll show both.

Plan: Write out the per-i iteration counts → sum them as an arithmetic series → get the closed form → reduce to Big-O → sanity-check with numbers and the "half-a-square" geometric picture.

First principles: total work = sum of inner counts

The total time of a nested loop is the sum of how many times the inner loop runs, across all outer iterations — not the product, because here the inner bound i changes every outer pass.

CODE
for i = 0 .. n-1:        # outer runs n times
    for j = 0 .. i:      # inner runs (i + 1) times  (inclusive j <= i)
        <O(1) work>
When i = 0, inner runs 1 time.
When i = 1, inner runs 2 times.
…
When i = n-1, inner runs n times.

So the total number of body executions is:

T(n) = 1 + 2 + 3 + … + n = Σ_{k=1}^{n} k.

Deriving the closed form (the arithmetic series)

The sum 1 + 2 + … + n is the classic arithmetic series. Pair the first and last terms: (1+n) + (2+(n-1)) + …, each pair sums to n+1, and there are n/2 such pairs:

T(n) = n(n+1)/2.

(If the inner bound is exclusive, for j = 0 .. i-1, the counts are 0,1,2,…,n-1 and the sum is n(n-1)/2 — differs only in the lower-order term.)

From exact count to complexity

n(n+1)/2 = n²/2 + n/2. In Big-O we (1) drop the lower-order term n/2 and (2) drop the constant factor ½:

Total time complexity = Θ(n²).

The key intermediate-level insight: even though the inner loop only goes up to i (so on average it runs about n/2 times, not n), the work is still n × n/2 = n²/2, which is the same growth class as a full n × n double loop. The "triangle" of work is exactly half the square, and half of a quadratic is still quadratic. Cutting the constant does not buy you a better complexity class.

Sanity checks

Numeric check: for n = 100, n(n+1)/2 = 100·101/2 = 5,050. A full square loop would do 100·100 = 10,000. The triangle is ~half — consistent with n²/2. For n = 1000 → 500,500 ≈ n²/2 = 500,000. Doubling n roughly quadruples the count (500,500 → ~2,001,000), the fingerprint of quadratic growth.

Geometric check: imagine an n × n grid; the loop visits the cells on or below the diagonal — a triangle with ≈ n²/2 cells. The area under the line f(i) = i from 0 to n is ∫ i di = n²/2, matching the sum.

Space complexity

Nothing is allocated per iteration — only the two loop counters and maybe an accumulator. So auxiliary space = O(1). (Time and space are analyzed separately; a quadratic-time loop can be constant-space.)

Where this pattern shows up

This exact triangular sum is the comparison count of simple quadratic sorts (selection/bubble/insertion), of "compare every pair (i, j) with j > i" problems (e.g. counting pairs, brute-force closest-pair, computing an upper-triangular distance matrix), and of building a triangular table. Recognizing "each element paired with the ones after it → n(n+1)/2 pairs → Θ(n²)" is a reusable pattern.

#### Common mistakes
/ follow-ups
Saying O(n²) but claiming it's "faster because inner only goes to i." It's the same Θ(n²) class; only the constant ½ differs. Don't mislabel that as sub-quadratic.
Saying it's O(n) because "the inner loop is small at first." No — you must sum; the average size is n/2, giving n².
Multiplying n × n = n² and moving on. That gives the right class here by luck, but the reasoning is wrong; if the interviewer asks for the exact count you need n(n+1)/2.
Forgetting the body cost. If the inner body were O(n) itself (e.g., a slice copy), total would be Σ i · n → n³.

#### Follow-up
they may ask: "What if the inner loop went j = i .. n?" That's the complement triangle, also n(n+1)/2 → Θ(n²). "What if inner ran n/i times?" → harmonic sum → Θ(n log n), a genuinely different class.

Strong-answer summary: total iterations = 1+2+…+n = n(n+1)/2 ≈ n²/2 (arithmetic series), which drops to Θ(n²) time and O(1) space. The one-line takeaway an interviewer wants: a triangular loop is half a square, and half a quadratic is still quadratic.

**Comparison:**

```
Bound style	Inner count per i	Total iterations	Big-O
j = 0 .. i (inclusive)	i + 1	n(n+1)/2	Θ(n²)
j = 0 .. i-1 (exclusive)	i	n(n-1)/2	Θ(n²)
full square j = 0 .. n	n	n²	Θ(n²)
Metric	Result
Exact iteration count	n(n+1)/2 = n²/2 + n/2
Time complexity	Θ(n²)
Auxiliary space	O(1)
Relation to square loop	~half the work, same class
```

**Code:**

```python
# The loop in the question (inclusive j <= i).
def triangular(n):
    count = 0
    for i in range(n):            # i = 0 .. n-1  (outer n times)
        for j in range(i + 1):    # j = 0 .. i    (inner i+1 times)
            count += 1            # O(1) body
    return count                  # == n*(n+1)//2

# Verification: closed form matches the loop for any n.
for n in [1, 5, 10, 100, 1000]:
    assert triangular(n) == n * (n + 1) // 2
# -> all pass; total iterations = n(n+1)/2  ->  Theta(n^2)
```

**Example input / output:**

```

Dry run for n = 5 (inclusive, inner runs i+1 times):

CODE
Copy
i = 0 -> inner runs 1 time   (running total 1)
i = 1 -> inner runs 2 times  (running total 3)
i = 2 -> inner runs 3 times  (running total 6)
i = 3 -> inner runs 4 times  (running total 10)
i = 4 -> inner runs 5 times  (running total 15)
total = 1+2+3+4+5 = 15 = 5*6/2  ->  Theta(n^2)

Growth vs. a full square loop:

CODE
Copy
 n    | triangle n(n+1)/2 | square n*n | ratio
 10   | 55                | 100        | ~0.55
 100  | 5,050             | 10,000     | ~0.51
 1000 | 500,500           | 1,000,000  | ~0.50   (approaches 1/2)
Doubling n -> count ~4x  ->  quadratic.
```

**Flow:**

1. for i in 0..n-1
2. inner runs (i+1) times
3. sum counts: 1 + 2 + ... + n
4. arithmetic series = n(n+1)/2
5. = n²/2 + n/2
6. drop n/2 and factor 1/2
7. Θ(n²) time, O(1) space

