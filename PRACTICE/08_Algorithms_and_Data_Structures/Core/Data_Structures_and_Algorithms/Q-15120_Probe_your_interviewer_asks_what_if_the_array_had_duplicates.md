---
type: "practice"
difficulty: "Deep"
---

# Probe: your interviewer asks 'what if the array had duplicates' — how does that change your binary search?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Inverted Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— decoding the probe

The interviewer asks: "what if the array had duplicates — how does that change your binary search?" This is a follow-up probe, not a fresh problem. It almost always comes after one of two setups, and a strong candidate names both because the answer differs:

Plain sorted array + duplicates — e.g., "find target" or "find the first/last occurrence of target." Here duplicates don't break binary search; they change what you return and force you to keep searching after a match.
Rotated sorted array + duplicates — e.g., "search in a rotated sorted array that may contain duplicates" (LeetCode 81). Here duplicates genuinely break the O(log n) guarantee and can degrade to O(n). This is the case the probe is usually fishing for at Amazon/Google/Microsoft SDE-2/3.

So the honest, senior answer is: "It depends on the array shape. Let me split it." I'll clarify with the interviewer which they mean, then cover both. The interviewer is probing: (a) do I understand why binary search works (the invariant), (b) do I know duplicates can silently break the "discard half" decision in a rotated array, and (c) can I state the exact complexity change and the correct fix (linear scan of the ambiguous boundary).

Clarifying questions: Is the array rotated or plain sorted? Do we want any index, the first index, the last index, or a count? Are we searching a value, or doing a monotonic-predicate binary search?

Case 1 — Plain sorted array with duplicates

Binary search still works perfectly; the array is still monotonic non-decreasing, so the "if a[mid] < target go right, else go left" decision is still valid. Duplicates only matter when you want a boundary:

First occurrence (lower bound): when a[mid] == target, don't stop — record the index and keep searching the left half (hi = mid - 1) to find an earlier one.
Last occurrence (upper bound): on a match, keep searching the right half (lo = mid + 1).
Count of target: lastIndex - firstIndex + 1, or upper_bound - lower_bound.

Complexity stays O(log n) because each step still halves the range. The classic bug here is a naive binary search that stops at the first match it finds — with duplicates that returns an arbitrary index in the run, not the boundary the problem wanted. The fix is the "record and keep going" pattern, or the standard lower_bound/upper_bound half-open-interval templates.

Case 2 — Rotated sorted array with duplicates (the real trap)

In a rotated sorted array without duplicates (LeetCode 33), binary search runs in O(log n): at each mid, exactly one half [lo..mid] or [mid..hi] is guaranteed sorted, and by comparing a[lo], a[mid], a[hi] we can always tell which half is sorted and whether the target lies inside it — so we always discard half.

Duplicates break this decision. The critical case is a[lo] == a[mid] == a[hi]. Example: [1,1,1,0,1,1], lo=0, hi=5, mid=2 → all three are 1. Now we cannot tell whether the sorted portion is on the left or the right — the target 0 could be on either side. The comparison a[lo] <= a[mid] no longer distinguishes "left half is sorted" from "we're straddling the rotation with equal endpoints."

The standard fix: when a[lo] == a[mid] == a[hi], we cannot discard a half safely, so we shrink the window by one on each end (lo++; hi--) and continue. This is a linear fallback. In the worst case — an array of nearly all equal elements like [2,2,2,2,2,2,2] searching for 3 — we do this n times, giving worst-case O(n). Average case stays close to O(log n) when duplicates are sparse. Note lo++; hi-- cannot skip the answer: if a[lo]==a[mid]==a[hi] and we're looking for something equal to that value, we'd have already matched at mid; otherwise the discarded endpoints equal a[mid] and are not the target.

So the crisp statement to the interviewer:

Correctness: still correct, but you must add the a[lo]==a[mid]==a[hi] branch that shrinks by one; without it, the algorithm can wrongly discard the half containing the target and return "not found."
Complexity: best/average O(log n), worst-case O(n) — the log-time guarantee is lost. That loss is unavoidable: with all-equal elements you fundamentally cannot decide a half without inspecting elements linearly.

A related classic (LeetCode 154, "find minimum in rotated sorted array with duplicates") has the exact same wrinkle: when a[mid] == a[hi], you can't tell which side the minimum is on, so you do hi-- (safe because a[hi] is duplicated by a[mid], so dropping it can't lose the unique minimum) — again O(n) worst case.

The invariant lens (why this all hangs together)

Binary search works whenever there's a monotonic decision function over the index space: "is the answer to the left or right of mid?" must be answerable in O(1). Duplicates in a plain sorted array keep the decision function monotonic, so binary search survives; you only refine which boundary you report. Duplicates in a rotated array make the O(1) decision ambiguous at equal endpoints, so you fall back to shrinking the window — trading the log-time guarantee for correctness. Framing the answer around this invariant is what earns the "deep understanding" tick.

Summary — what a strong answer emphasizes
Split by array shape first. Plain sorted vs rotated behave very differently.
Plain sorted + duplicates: binary search stays O(log n); use lower/upper-bound "record and keep searching" to get first/last/count. Naive "stop on first match" is the bug.
Rotated + duplicates: the a[lo]==a[mid]==a[hi] case is undecidable, so shrink by one (lo++, hi--); correctness preserved but worst case degrades to O(n) — and that degradation is provably unavoidable.
Tie it to the monotonic-predicate invariant and state complexity precisely for each case.

#### Common mistakes
claiming duplicates always keep it O(log n) (false for rotated); forgetting the triple-equal branch (returns wrong "not found"); stopping at the first equal element when the boundary/count was wanted; and using lo <= hi vs lo < hi loop conditions inconsistently across the boundary templates.

**Comparison:**

```
Scenario	Does BS still work?	What changes	Complexity
Plain sorted, find any index	Yes	Nothing	O(log n)
Plain sorted, first/last/count	Yes	"Record match, keep searching that side" (lower/upper bound)	O(log n)
Rotated, no duplicates	Yes	One half always provably sorted	O(log n)
Rotated, with duplicates	Yes, with fix	Add a[lo]==a[mid]==a[hi] → lo++,hi-- fallback	O(log n) avg, O(n) worst
Rotated dups, find minimum (LC154)	Yes, with fix	a[mid]==a[hi] → hi--	O(log n) avg, O(n) worst
```

**Code:**

```python
# Case 1: plain sorted array with duplicates -> first / last occurrence (O(log n))
def first_occurrence(a, target):
    lo, hi, ans = 0, len(a) - 1, -1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == target:
            ans = mid          # record...
            hi = mid - 1       # ...but keep going LEFT for an earlier one
        elif a[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return ans

def last_occurrence(a, target):
    lo, hi, ans = 0, len(a) - 1, -1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == target:
            ans = mid
            lo = mid + 1       # keep going RIGHT for a later one
        elif a[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return ans

def count_target(a, target):
    f = first_occurrence(a, target)
    if f == -1:
        return 0
    return last_occurrence(a, target) - f + 1

# Case 2: rotated sorted array WITH duplicates -> exists? (LeetCode 81)
# Best/avg O(log n), worst O(n) due to the triple-equal fallback.
def search_rotated_with_dups(a, target):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == target:
            return True
        # The ambiguous case: can't tell which half is sorted -> shrink by one.
        if a[lo] == a[mid] == a[hi]:
            lo += 1
            hi -= 1
        elif a[lo] <= a[mid]:            # left half [lo..mid] is sorted
            if a[lo] <= target < a[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        else:                            # right half [mid..hi] is sorted
            if a[mid] < target <= a[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return False
```

**Example input / output:**

```text
Case 1 (plain sorted, duplicates):
a = [1, 2, 2, 2, 3, 4, 4], target = 2
  first_occurrence -> 1   (index of first 2)
  last_occurrence  -> 3   (index of last 2)
  count_target     -> 3
Naive "stop on first match" would return 2 (arbitrary), which is wrong for a boundary query.

Case 2 (rotated + duplicates) — the trap trace:
a = [1, 1, 1, 0, 1, 1], target = 0
  lo=0,hi=5,mid=2 -> a[0]=a[2]=a[5]=1  => AMBIGUOUS => lo=1, hi=4
  lo=1,hi=4,mid=2 -> a[2]=1 != 0; a[1]=a[2]=a[4]=1 => AMBIGUOUS => lo=2, hi=3
  lo=2,hi=3,mid=2 -> a[2]=1; a[2]=1 <= a[2]=1 (left sorted), 1<=0<1? no => lo=3
  lo=3,hi=3,mid=3 -> a[3]=0 == target => return True
Without the triple-equal branch, step 1 would have mis-decided and returned False.

Worst case: a = [2,2,2,2,2,2,2], target = 3
  every step hits triple-equal -> lo++,hi-- -> ~n/2 iterations -> O(n), returns False.
```

**Flow:**

1. Plain sorted
2. Rotated sorted
3. yes
4. no
5. Probe: duplicates in binarysearch
6. Array shape?
7. BS still O log n
8. Want boundary? uselower/upper bound:record match, keep searchingthat side
9. a[lo]==a[mid]==a[hi]?
10. Ambiguous: cannot discardhalflo++, hi-- -> linear fallback ->O n worst
11. Exactly one half is sorteddiscard the other -> O log nstep

