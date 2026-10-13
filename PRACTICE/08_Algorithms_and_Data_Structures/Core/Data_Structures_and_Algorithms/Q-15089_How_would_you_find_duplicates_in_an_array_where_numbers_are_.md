---
type: "practice"
difficulty: "Intermediate"
---

# How would you find duplicates in an array where numbers are in the range 1 to n?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Combinations](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Combinations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Given an array of n (or roughly n) integers where every value is guaranteed to lie in the range [1, n], find the numbers that appear more than once. The tight, non-obvious phrasing — "range 1 to n" — is the whole point: it is a strong hint that the values themselves can be used as array indices. The interviewer is probing whether you (a) reach a correct hash-set solution quickly, and (b) recognize the constraint that unlocks an O(n) time, O(1) extra space in-place trick.

Clarifying questions a strong candidate asks first:

Is the array read-only, or am I allowed to mutate it? (Mutation permits the O(1)-space trick.)
Can a number appear more than twice? (Changes whether I return distinct duplicates or every extra occurrence.)
Is the array exactly length n with values in [1, n], so it's a "n slots, n values" pigeonhole setup? Or length n+1 (the classic single-duplicate variant)?
Should the output be sorted / distinct?

I'll assume: array of length n, values in [1, n], mutation allowed unless told otherwise, and I must return the distinct set of duplicated values.

Plan: Start from the brute force, improve with a hash set (the answer most interviews accept), then present the space-optimal in-place index-marking / cyclic-sort approach that the constraint is designed to elicit. Discuss trade-offs and edge cases.

Step 1 — Brute force (establish a baseline)

Compare every pair: for each i, scan j > i and record arr[i] if it matches any later element. This is O(n^2) time, O(1) space. Correct but too slow — never leave the interview here; state it and move on.

Step 2 — Hash set (the clean, always-acceptable answer)

Walk once through the array, keeping a seen set. Before inserting each value, check if it's already present; if so, it's a duplicate. This is O(n) time, O(n) space. It works regardless of the "1 to n" constraint and even if the array is read-only — so it's the safe, universal answer. Most interviewers accept it fully; then they'll ask, "can you do it without extra space?" — which is where the constraint matters.

Frequency array/Counter is a close cousin: count occurrences, emit anything with count ≥ 2. Also O(n)/O(n).

Step 3 — Exploit "values in [1, n]": in-place index marking (O(1) extra space)

Because every value v is a valid index (after mapping v → v-1 for 0-based arrays), the array can double as its own hash table. Two standard techniques:

(a) Negation marking (mutation allowed, all values positive). For each element, let idx = abs(arr[i]) - 1. If arr[idx] is already negative, then the value idx+1 has been seen before → it's a duplicate. Otherwise flip arr[idx] negative to mark "seen". Because we use abs() when reading, prior flips don't corrupt future reads. This is O(n) time, O(1) extra space. It mutates the array (you can restore signs in a second pass if the caller needs the original).

(b) Cyclic sort (place each value at its home index). Repeatedly swap arr[i] to position arr[i]-1 until either i is settled or a swap would send a value onto an equal value already sitting at its home — that collision reveals a duplicate. After the pass, any position i where arr[i] != i+1 also flags a misplaced/duplicate value. O(n) time, O(1) space. It's more general (handles missing numbers too) but trickier to code correctly under pressure.

I'd present negation as the primary optimal answer — it's shorter and less error-prone.

Edge cases & correctness
Empty array → return empty.
The negation trick assumes all values are positive (guaranteed here since range is [1, n]) — that's exactly why it's applicable. If zeros or negatives were possible, negation breaks and you'd fall back to the hash set.
To emit distinct duplicates when a value appears 3+ times, guard so you record it only on the first detection (negation naturally does this because after the first flip it stays negative — but a value appearing 4 times would be detected on the 2nd, 3rd, 4th occurrences; dedupe by only appending the first time you see it already-marked, or collect into a set).
Read-only array + O(1) space is impossible in general — you must relax one constraint. Say this out loud; it shows maturity.
Complexity summary

#### Approach
Time	Extra space	Mutates?	When to use
Brute force	O(n²)	O(1)	No	Never in practice; baseline only
Hash set / Counter	O(n)	O(n)	No	Default; read-only ok; universal
Negation marking	O(n)	O(1)	Yes	Values in [1,n], mutation allowed
Cyclic sort	O(n)	O(1)	Yes	Also finds missing numbers
What a strong answer emphasizes

Recognize that "1 to n" is the pigeonhole/index-as-hash signal, quote the hash-set O(n)/O(n) answer immediately, then upgrade to O(1) space via negation, and be explicit about the trade-off you're making (mutation vs. extra memory) and the edge case that breaks each. Common mistakes: forgetting abs() when reading (so a previously flipped sign is misread), double-reporting a value that occurs 3+ times, or claiming O(1) space while the array is read-only.

**Comparison:**

```
Approach	Time	Extra space	Mutates?	Works if read-only?
Brute force pairs	O(n²)	O(1)	No	Yes
Hash set / Counter	O(n)	O(n)	No	Yes
In-place negation	O(n)	O(1)	Yes	No
Cyclic sort	O(n)	O(1)	Yes	No
```

**Code:**

```python
from typing import List

# Approach A: hash set — O(n) time, O(n) space, works even if read-only
def find_duplicates_hashset(arr: List[int]) -> List[int]:
    seen, dups = set(), set()
    for v in arr:
        if v in seen:
            dups.add(v)
        else:
            seen.add(v)
    return sorted(dups)

# Approach B: in-place negation — O(n) time, O(1) extra space
# Requires values in [1, n] and mutation allowed. Returns DISTINCT duplicates.
def find_duplicates_inplace(arr: List[int]) -> List[int]:
    dups = []
    for i in range(len(arr)):
        idx = abs(arr[i]) - 1          # map value -> index; ignore prior sign flips
        if arr[idx] < 0:               # already marked -> duplicate
            dups.append(idx + 1)
        else:
            arr[idx] = -arr[idx]       # mark "seen"
    # (optional) restore original array: arr[i] = abs(arr[i]) for all i
    return dups

if __name__ == "__main__":
    print(find_duplicates_hashset([4, 3, 2, 7, 8, 2, 3, 1]))  # [2, 3]
    print(find_duplicates_inplace([4, 3, 2, 7, 8, 2, 3, 1]))  # [2, 3]
```

**Example input / output:**

```

Dry run of the negation trick on arr = [4, 3, 2, 7, 8, 2, 3, 1] (n = 8, 0-based):

CODE
Copy
i=0 v=4  idx=3  arr[3]=7>0  -> mark  arr=[4, 3, 2,-7, 8, 2, 3, 1]
i=1 v=3  idx=2  arr[2]=2>0  -> mark  arr=[4, 3,-2,-7, 8, 2, 3, 1]
i=2 v=-2 idx=1  arr[1]=3>0  -> mark  arr=[4,-3,-2,-7, 8, 2, 3, 1]
i=3 v=-7 idx=6  arr[6]=3>0  -> mark  arr=[4,-3,-2,-7, 8, 2,-3, 1]
i=4 v=8  idx=7  arr[7]=1>0  -> mark  arr=[4,-3,-2,-7, 8, 2,-3,-1]
i=5 v=2  idx=1  arr[1]=-3<0 -> DUP 2 (append 2)
i=6 v=-3 idx=2  arr[2]=-2<0 -> DUP 3 (append 3)
i=7 v=-1 idx=0  arr[0]=4>0  -> mark  arr=[-4,-3,-2,-7,8,2,-3,-1]

Output: [2, 3]

Second example: input [1, 1, 2] -> output [1]. Input [1, 2, 3] -> output [].
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Yes
6. No
7. Start: array, values in 1..n
8. Read-only?
9. Hash set / Counter: O(n) time,O(n) space
10. Need O(1) extra space?
11. For each i: idx = abs(arr[i]) - 1
12. arr[idx] < 0 ?
13. Record idx+1 as duplicate
14. arr[idx] = -arr[idx] mark seen
15. Next i
16. Return duplicates

