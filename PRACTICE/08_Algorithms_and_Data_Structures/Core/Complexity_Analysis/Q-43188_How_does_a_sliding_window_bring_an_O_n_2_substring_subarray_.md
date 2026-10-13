---
type: "practice"
difficulty: "Intermediate"
---

# How does a sliding window bring an O(n^2) substring/subarray problem down to O(n)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive vs Iterative Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Recursive_vs_Iterative_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Explain how and why the sliding-window technique reduces a substring/subarray problem from O(n²) (recompute every window from scratch) to O(n) (slide a window and update incrementally). The interviewer wants to see that you understand the core mechanism — reusing the previous window's work by adding the incoming element and removing the outgoing one, instead of recomputing — and the precondition that makes it valid: the window's validity/answer must be updatable in O(1) amortized as the boundaries move, and both boundaries only move forward (never backtrack). A practitioner answer distinguishes fixed-size vs variable-size windows, gives a canonical example with a trace, and states where it applies.

#### Assumptions
/ clarifying questions:

Fixed window size k, or a variable window satisfying some constraint (e.g., "longest substring with no repeats", "smallest subarray with sum ≥ S")?
Contiguous subarray/substring (required — sliding window needs contiguity)?
Can I maintain the window's state (a running sum, a char-count map, a max) in O(1) per move?

Plan: show why brute force is O(n²), then show the two window styles and the incremental-update idea, prove the O(n) bound via the "each pointer advances at most n times" argument, and give a worked trace.

Why brute force is O(n²)

Brute force enumerates every contiguous window and recomputes its property. For a subarray-sum problem:

CODE
for start in 0..n-1:
    for end in start..n-1:
        window = a[start..end]
        recompute property over window   # e.g., sum, or distinct-count

There are ~n²/2 windows, and recomputing each from scratch is where the waste is: window [start..end] and [start..end+1] overlap almost entirely, yet brute force throws away all the shared work and re-adds the same elements repeatedly. That redundancy is the O(n²) (or even O(n³) if the inner recompute is itself O(n)).

The sliding-window idea: reuse, don't recompute

The window is a contiguous range [left, right]. Instead of rebuilding it, you maintain a running state and update it in O(1) as the window moves:

Expand: move right forward → add a[right] to the state (sum += a[right], or increment count[a[right]]).
Contract: move left forward → remove a[left] from the state (sum -= a[left], or decrement count[a[left]]).

Each element is added once (when right passes it) and removed at most once (when left passes it). Since left and right each advance from 0 to n and never move backward, total work is bounded by 2n operations → O(n) time. Extra space is O(1) for a running sum, or O(k) / O(alphabet) for a frequency map — small and independent of the number of windows.

This is exactly the same "monotone pointer movement" engine as the fast/slow two-pointer technique: the reason it's linear is that the boundaries only sweep forward once.

Two window styles

1. Fixed-size window (size k). Slide a window of constant width across the array. Compute the property once for the first window in O(k), then for each subsequent position do one add + one remove in O(1). Total O(n). Classic: "max sum of any subarray of length k" — windowSum += a[right] - a[left] as you slide.

2. Variable-size window. The window grows/shrinks to satisfy a constraint. Two loops that look nested but aren't O(n²):

CODE
left = 0
for right in 0..n-1:
    add a[right] to window
    while window is INVALID:      # shrink until valid again
        remove a[left]; left += 1
    update answer with current window

The inner while seems to add nesting, but left only ever moves forward across the whole array — the total number of left++ executions over the entire run is at most n. So the amortized cost is O(1) per step and the whole thing is O(n). Examples: "longest substring without repeating characters," "smallest subarray with sum ≥ S," "longest substring with at most k distinct characters."

The precondition for the variable window: the constraint must be monotone — if a window is invalid, extending right can't fix it, and shrinking from left is the only cure; and once valid, shrinking further from left keeps a valid (possibly better) window discoverable. This monotonicity is what guarantees left never needs to move backward, which is what preserves O(n). If the property isn't monotone (e.g., "subarray with sum exactly T" over arrays with negatives), a plain sliding window is invalid and you need a different tool (prefix-sum + hash map).

Worked example (longest substring without repeating characters)

State = a set/map of chars in the current window. Expand right; when the incoming char is already inside, shrink left past its previous occurrence; track the max width. Each char enters and leaves the window once → O(n) time, O(min(n, alphabet)) space.

Summary

Sliding window turns O(n²) into O(n) by carrying the window's answer forward incrementally — one add for the incoming element, one remove for the outgoing one — instead of recomputing each overlapping window from scratch. Both boundaries only sweep forward once, so total work is O(n); extra space is O(1) or O(alphabet). A strong answer stresses the incremental-update mechanism, distinguishes fixed vs. variable windows, and gives the amortization argument for why the inner while-shrink loop is still linear. Common mistakes: recomputing the whole window inside the loop (defeats the purpose, back to O(n²)); using it on non-contiguous or non-monotone problems; forgetting to shrink; off-by-one on window bounds. Follow-up probes: "Prove the inner loop is amortized O(1)," "Why doesn't this work for arrays with negative numbers and an exact-sum target?" (constraint isn't monotone → use prefix sums + hashing), "Fixed vs. variable window — how do you decide?" (fixed = size given; variable = grow/shrink to meet a constraint and optimize width/count).

**Comparison:**

```
	Brute force (all windows)	Sliding window
Time	O(n²) (or O(n³) if inner recompute is O(n))	O(n)
Extra space	O(1)	O(1) (running sum) or O(alphabet)/O(k) (freq map)
Core idea	recompute each window from scratch	incremental add-one / remove-one update
Boundary movement	nested full re-scan	left & right each advance ≤ n total (monotone)
Precondition	none	contiguous window; constraint monotone; O(1) state update
Fixed vs variable	—	fixed = given size k; variable = grow/shrink to meet constraint
Fails when	never (just slow)	non-contiguous, non-monotone (e.g. exact-sum with negatives)
```

**Code:**

```python
# Fixed-size window: max sum of any subarray of length k. O(n) time, O(1) space.
def max_sum_k(a, k):
    s = sum(a[:k])                 # first window, O(k)
    best = s
    for right in range(k, len(a)): # slide: add incoming, drop outgoing -> O(1) each
        s += a[right] - a[right - k]
        best = max(best, s)
    return best

# Variable-size window: longest substring without repeating chars. O(n) time.
def longest_unique(s):
    last = {}                      # char -> last index seen
    left = 0
    best = 0
    for right, c in enumerate(s):
        if c in last and last[c] >= left:
            left = last[c] + 1     # shrink: jump past the duplicate
        last[c] = right
        best = max(best, right - left + 1)
    return best

# Variable-size window: smallest subarray with sum >= S (positive nums). O(n).
def min_len_subarray(a, S):
    left = 0; total = 0; best = float('inf')
    for right in range(len(a)):
        total += a[right]              # expand
        while total >= S:             # shrink while still valid
            best = min(best, right - left + 1)
            total -= a[left]; left += 1
    return 0 if best == float('inf') else best
```

**Example input / output:**

```text
Fixed window — a = [2, 1, 5, 1, 3, 2], k = 3:
  window [2,1,5] sum=8 (best=8)
  slide -> [1,5,1] sum = 8 + 1 - 2 = 7
  slide -> [5,1,3] sum = 7 + 3 - 1 = 9  (best=9)
  slide -> [1,3,2] sum = 9 + 2 - 5 = 6
  Answer: 9. Each slide is one add + one subtract -> O(n).

Variable window — s = "abcabcbb":
  right=a: win="a" (len 1)
  b: "ab"(2)  c: "abc"(3, best=3)
  a: dup -> left jumps to 1, "bca"(3)
  b: dup -> left to 2, "cab"(3)
  c: dup -> left to 3, "abc"(3)
  b: dup -> left to 5, "cb"(2)
  b: dup -> left to 7, "b"(1)
  Answer: 3 ("abc"). Every char added once, removed once -> O(n).

Smallest subarray sum >= 7 — a = [2,3,1,2,4,3]:
  expands then shrinks; best window = [4,3] -> length 2.
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. right = 0 .. n-1
6. add a[right] to window state
7. window valid?
8. remove a[left]; left++
9. update answer with window
10. right < n-1?
11. return answer

