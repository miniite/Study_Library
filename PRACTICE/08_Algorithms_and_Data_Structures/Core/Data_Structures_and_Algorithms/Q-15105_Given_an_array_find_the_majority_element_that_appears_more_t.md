---
type: "practice"
difficulty: "Deep"
---

# Given an array, find the majority element that appears more than n/2 times (Boyer-Moore).

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Majority Element](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Majority_Element.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to find the majority element of an array — the value that appears strictly more than ⌊n/2⌋ times — and it names Boyer-Moore as the expected technique. At "deep understanding" level the interviewer is not just checking that I know the trick; they want me to (1) get the definition and its consequences right, (2) explain why Boyer-Moore voting works (the invariant), not just recite it, (3) hit O(n) time and O(1) space, and (4) handle the edge case that many candidates miss.

Clarifying assumptions I'd state up front:

"More than n/2" is the strict majority. This is important because it guarantees the element is unique (there can be at most one element with >n/2 count) and that Boyer-Moore's single-pass candidate is correct.
Is a majority element guaranteed to exist? If the interviewer says yes, a single pass suffices. If not guaranteed, I must add a verification pass to confirm the candidate actually crosses n/2, otherwise I'd return a wrong answer for inputs like [1,2,3]. A deep answer always mentions this. I'll code the safe version.
First principles: why brute force is unsatisfying

The naive approaches are: (a) count every element with a hash map — O(n) time but O(n) space; or (b) sort and take the middle element a[n/2] — because if an element occupies more than half the slots, it must straddle the median index — but that's O(n log n). Both work, but the interviewer named Boyer-Moore because it beats them on space: O(n) time, O(1) extra space, single logical pass. That is the whole point of the algorithm.

The Boyer-Moore Voting idea

Think of it as pairwise annihilation. If every element could "cancel out" one element of a different value, then because the majority element has more than half the total count, it can never be fully cancelled — some copies of it always survive. Boyer-Moore implements this cancellation on the fly with just two variables:

candidate — the current guess for the majority element.
count — a running "lead" of the candidate over everything else seen so far.

Scan left to right:

If count == 0, adopt the current element as the new candidate (the previous candidate has been fully cancelled, so we restart with a fresh guess).
If the current element equals candidate, increment count (a vote for).
Otherwise decrement count (a vote against / a cancellation).

After one pass, candidate holds the majority element if one exists.

Why it works — the invariant (this is the "deep understanding" part)

The key invariant: at every step, count equals the number of candidate-copies remaining after cancelling each with one non-candidate copy in the prefix processed so far. Equivalently, count = (occurrences of candidate) − (occurrences of everything else) within the current "run," and it never goes negative — when it would, we reset and pick a new candidate.

Now the correctness argument. Let M be the true majority element with count k > n/2, so all other elements together number n − k < n/2 < k. Every decrement of count (or a reset that discards a candidate) can be charged to cancelling one candidate against one different element. Since the non-M elements total fewer than k, they cannot cancel all k copies of M. So M cannot be permanently eliminated: whenever M is the candidate its surplus is never fully paid down, and even if the algorithm temporarily adopts some other candidate, that candidate is a minority element and will be cancelled to zero, after which M gets adopted again and finishes with count > 0. Hence the final candidate must be M. This argument relies on strict majority — if the max count were exactly n/2 it could be cancelled and the guarantee breaks, which is exactly why the problem says more than n/2.

The mandatory second pass

Boyer-Moore only finds a candidate. If a majority element isn't guaranteed to exist, the candidate could be a plurality-but-not-majority element (or garbage). So I do a verification pass: count actual occurrences of candidate and confirm it exceeds n/2; if not, report "no majority." This keeps the algorithm correct for arbitrary input while staying O(n) time / O(1) space. Skipping this is the single most common mistake — and a deep-understanding interviewer will feed [1,2,3] to catch it.

Complexity
Time: O(n) — two linear passes (find candidate, then verify), which is still O(n).
Space: O(1) — two scalar variables regardless of input size.
This is optimal: you must read every element at least once (any unread element could flip the answer), so Ω(n) time is a lower bound, and O(1) space is best possible.
Summary — what a strong answer emphasizes

State the strict >n/2 definition and its uniqueness consequence; give Boyer-Moore as O(n)/O(1); explain the cancellation invariant and why strict majority makes the surviving candidate correct; and always add the verification pass unless existence is guaranteed. Mention the alternatives (hashmap O(n) space, sort-and-take-middle O(n log n)) to show you chose Boyer-Moore deliberately for its O(1) space.

#### Follow-up
probes to be ready for: (1) Boyer-Moore generalization — find all elements appearing more than n/3 times (there can be at most two; track two candidates and two counters, then verify both). (2) Streaming — the algorithm is naturally single-pass and works on a stream, but you need a second pass or a guarantee to verify, which a pure stream can't always give. (3) Why not just sort? — correct but O(n log n) and mutates/needs the whole array. Handling the n/3 extension cleanly is the strongest signal of genuine understanding.

**Comparison:**

```
Approach	Time	Space	Notes
Hash-map count	O(n)	O(n)	Simple; loses on space
Sort, take a[n/2]	O(n log n)	O(1)*	Middle element is the majority if it exists (*O(n) if not in place)
Boyer-Moore voting	O(n)	O(1)	Optimal; needs verify pass unless existence guaranteed
Randomized pick + verify	O(n) expected	O(1)	Pick random index, check if it's majority; works because >50% chance
```

**Code:**

```

from typing import List, Optional

def majority_element(nums: List[int]) -> Optional[int]: """Boyer-Moore voting. Returns the element appearing > n/2 times, or None if no such element exists. O(n) time, O(1) space.""" # Pass 1: find a candidate. candidate, count = None, 0 for x in nums: if count == 0: candidate = x count += 1 if x == candidate else -1

CODE
Copy
# Pass 2: verify (skip only if majority is GUARANTEED to exist).
if candidate is not None and nums.count(candidate) > len(nums) // 2:
    return candidate
return None
CODE
Copy

```java
// Same algorithm in Java, verification included.
static Integer majorityElement(int[] nums) {
    Integer candidate = null;
    int count = 0;
    for (int x : nums) {
        if (count == 0) candidate = x;
        count += (candidate != null && x == candidate) ? 1 : -1;
    }
    int occ = 0;
    if (candidate != null) for (int x : nums) if (x == candidate) occ++;
    return (candidate != null && occ > nums.length / 2) ? candidate : null; // null = no majority
}
PYTHON
Copy
# Follow-up: elements appearing > n/3 times (at most two). Two candidates.
def majority_n_over_3(nums):
    c1 = c2 = None; n1 = n2 = 0
    for x in nums:
        if x == c1: n1 += 1
        elif x == c2: n2 += 1
        elif n1 == 0: c1, n1 = x, 1
        elif n2 == 0: c2, n2 = x, 1
        else: n1 -= 1; n2 -= 1
    return [c for c in {c1, c2}
            if c is not None and nums.count(c) > len(nums)//3]
```

**Example input / output:**

```text
Example 1 (majority exists):
  input:  [2, 2, 1, 1, 1, 2, 2]        n = 7, need count > 3
  output: 2   (appears 4 times > 3)

Dry run of pass 1:
  x=2  count 0 -> candidate=2, count=1
  x=2  ==cand  -> count=2
  x=1  !=cand  -> count=1
  x=1  !=cand  -> count=0
  x=1  count 0 -> candidate=1, count=1
  x=2  !=cand  -> count=0
  x=2  count 0 -> candidate=2, count=1
  final candidate = 2 ; verify: 2 occurs 4 times > 3  -> return 2  ✔

Example 2 (NO majority — why the second pass matters):
  input:  [1, 2, 3]        n = 3, need count > 1
  pass 1 candidate ends as 3, but 3 occurs only 1 time (not > 1)
  output: None            (a single-pass-only solution would wrongly return 3)

Example 3 (single element):
  input:  [7]  -> candidate 7, occurs 1 > 0  -> return 7
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. yes
6. no
7. yes
8. no
9. Start: candidate=None,count=0
10. More elements?
11. Verify: count occurrences ofcandidate
12. count == 0?
13. candidate = x, count = 1
14. x == candidate?
15. count += 1
16. count -= 1
17. occurrences > n/2?
18. return candidate
19. return None / no majority

