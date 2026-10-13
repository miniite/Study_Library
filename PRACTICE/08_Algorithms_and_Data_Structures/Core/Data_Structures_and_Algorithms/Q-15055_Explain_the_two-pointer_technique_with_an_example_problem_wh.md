---
type: "practice"
difficulty: "Intermediate"
---

# Explain the two-pointer technique with an example problem where it beats brute force.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Algorithms](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting_Algorithms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: (1) Explain the two-pointer technique conceptually, and (2) pick a concrete problem where it clearly beats brute force, and show why (the complexity delta and the invariant that makes it correct). The interviewer wants to see that you understand two-pointer as a family of patterns — not one trick — and that you can justify correctness, not just recite code.

Clarifying questions: Which flavor do they want — opposite-ends (converging), same-direction (fast/slow), or two-sequence merge? I'll cover the pattern space, then anchor on the canonical "pair with target sum in a sorted array" as the beats-brute-force example, and mention the flavors.

Plan: Define the technique and its variants → state the enabling precondition (usually sortedness or monotonicity) → pick "two-sum on sorted array" → prove the O(n) two-pointer beats O(n²) brute force with a correctness argument → dry run → table of other classic applications → pitfalls/follow-ups.

What the two-pointer technique is

Two-pointer means maintaining two indices into one (or two) sequences and moving them under a rule so that you avoid re-examining combinations you can already rule out. It converts many O(n²) "check all pairs" problems into a single O(n) pass. There are three common flavors:

Opposite ends / converging: left starts at 0, right at n-1, and they move toward each other based on a comparison. Works on sorted arrays. Examples: pair with target sum, container-with-most-water, valid palindrome, reverse array.
Same direction / fast-slow: both start on the left; one advances faster or conditionally. Examples: remove duplicates in-place, move zeros, cycle detection (Floyd's tortoise-and-hare), find nth-from-end in a linked list. (Sliding window is a specialized same-direction two-pointer where the pointers bound a window.)
Two sequences: one pointer per array, advancing the one that "lags". Examples: merge two sorted arrays, intersection of sorted arrays, is-subsequence.

The unifying idea: each pointer moves forward-only (or converges) so total movement is O(n), and at each step a comparison lets you discard a whole set of candidates without checking them individually.

The enabling precondition

The reason a pointer can safely skip candidates is usually sortedness or some monotonic structure. If the data isn't sorted and the problem needs it, you either sort first (O(n log n), which may still beat O(n²)) or the two-pointer approach doesn't apply and you use a hashmap instead. Stating this precondition is what separates a deep answer from a shallow one.

Beats-brute-force example: pair with target sum in a sorted array

Problem: Given a sorted array and a target T, return indices of two elements summing to T (or whether such a pair exists).

Brute force: check every pair (i, j) → O(n²) time, O(1) space.

Two-pointer (converging): left = 0, right = n-1. Look at sum = a[left] + a[right].

If sum == T → found.
If sum < T → the sum is too small; the only way to increase it is to move left right (to a larger value). Moving right left would only make it smaller. So left++.
If sum > T → too big; must decrease → right--.

Each step moves exactly one pointer inward, so at most n steps → O(n) time, O(1) space.

Why it's correct (the discard argument): When sum < T and we do left++, we claim we can safely never revisit the old left with any right' < right. Proof: for the current left, a[left] + a[right] is the largest sum achievable with this left (since right is the largest remaining index and the array is sorted). If even that largest sum is < T, then a[left] paired with any smaller element can't reach T either. So discarding all pairs (left, *) is safe. Symmetric argument for sum > T and right--. This eliminates a whole row/column of the pair matrix per step — that's the source of the speedup. This monotonic reasoning is exactly what the interviewer is listening for.

Complexity comparison
Brute force: O(n²) time, O(1) space.
Two-pointer (array already sorted): O(n) time, O(1) space.
If the array is unsorted and you must sort: O(n log n) overall — still beats O(n²), though a hashmap gives O(n) time / O(n) space and doesn't need sorting. Good candidates mention this trade-off: two-pointer wins on space (O(1)) and when the input is already sorted; hashmap wins on time for unsorted input.
Dry run

Sorted [1, 3, 4, 5, 7, 11], target T = 9:

left=0 (1), right=5 (11) → sum 12 > 9 → right--
left=0 (1), right=4 (7) → sum 8 < 9 → left++
left=1 (3), right=4 (7) → sum 10 > 9 → right--
left=1 (3), right=3 (5) → sum 8 < 9 → left++
left=2 (4), right=3 (5) → sum 9 == 9 → found (indices 2,3)

Four elements were never paired against each other, yet we're certain no earlier pair summed to 9 — the discard argument guarantees it.

Other classic wins (mention breadth)
Container with most water: O(n) vs O(n²); move the pointer at the shorter wall inward, because the shorter wall bounds the area and moving it is the only chance to improve.
Remove duplicates from sorted array in place: O(n), O(1) — slow pointer marks the write position, fast pointer scans.
Cycle detection (Floyd's): fast/slow pointers, O(n) time, O(1) space vs O(n) space for a visited-set.
3-sum: sort, then fix one element and run converging two-pointer for each → O(n²), beating O(n³) brute force.
Pitfalls and follow-up probes
Applying converging two-pointer to an unsorted array — invalid; the discard argument relies on order. Sort first or use a hashmap.
Infinite loops / off-by-one when pointers cross: loop condition must be left < right (strictly), and handle "no pair found".
Duplicates in problems like 3-sum: skip equal neighbors to avoid duplicate triples.
Probe: "What if the array is unsorted and you need indices in the original order?" → hashmap of value→index in one pass, O(n) time/space; you lose the O(1)-space advantage but avoid the sort.
Summary — what a strong answer emphasizes

Two-pointer is a family (converging, fast/slow, two-sequence) that trades O(n²) pair enumeration for a single O(n)/O(1)-space pass by moving forward-only/converging pointers and discarding whole candidate sets at each comparison. The canonical beats-brute-force example is pair-with-target on a sorted array: O(n) vs O(n²), justified by the "largest achievable sum with this pointer" discard argument. Always state the sortedness/monotonicity precondition and the two-pointer-vs-hashmap trade-off (space vs. needing sorted input) — that judgment is what earns the SDE-2/3 signal.

**Comparison:**

```
Flavor	Pointer motion	Precondition	Example	Beats
Converging (opposite ends)	move inward on comparison	sorted array	pair sum = T, container water, palindrome	O(n²)→O(n)
Fast/slow (same direction)	slow write / fast scan	in-place edit or cycle	remove dups, move zeros, Floyd cycle	O(n) space→O(1)
Two-sequence	advance the lagging pointer	both sorted	merge, intersection, is-subsequence	O(n·m)→O(n+m)
Approach for pair-sum	Time	Space	Needs sorted?
Brute force	O(n²)	O(1)	no
Two-pointer	O(n)	O(1)	yes
Sort + two-pointer	O(n log n)	O(1)	sorts it
Hashmap	O(n)	O(n)	no
```

**Code:**

```java
// Converging two-pointer: does a sorted array contain a pair summing to T?
// Returns the pair of indices, or {-1,-1}. O(n) time, O(1) space.
int[] twoSumSorted(int[] a, int T) {
    int left = 0, right = a.length - 1;
    while (left < right) {
        int sum = a[left] + a[right];
        if (sum == T) return new int[]{left, right};
        if (sum < T) left++;    // need a bigger sum -> discard smallest left
        else         right--;   // need a smaller sum -> discard largest right
    }
    return new int[]{-1, -1};
}

// Contrast: brute force, O(n^2) — checks every pair.
int[] twoSumBrute(int[] a, int T) {
    for (int i = 0; i < a.length; i++)
        for (int j = i + 1; j < a.length; j++)
            if (a[i] + a[j] == T) return new int[]{i, j};
    return new int[]{-1, -1};
}

// Same-direction flavor: remove duplicates from a sorted array in place, O(n)/O(1).
int removeDuplicates(int[] a) {
    if (a.length == 0) return 0;
    int slow = 0;                       // write index
    for (int fast = 1; fast < a.length; fast++)
        if (a[fast] != a[slow]) a[++slow] = a[fast];
    return slow + 1;                    // new length
}
```

**Example input / output:**

```

twoSumSorted([1, 3, 4, 5, 7, 11], T=9):

CODE
Copy
left=0(1) right=5(11) sum=12 >9  right--
left=0(1) right=4(7)  sum=8  <9  left++
left=1(3) right=4(7)  sum=10 >9  right--
left=1(3) right=3(5)  sum=8  <9  left++
left=2(4) right=3(5)  sum=9  ==9 -> return [2,3]

5 comparisons vs. up to 15 for brute force on n=6 (n(n-1)/2).

removeDuplicates([1,1,2,2,2,3]):

CODE
Copy
scan -> writes 1,2,3 ; returns length 3 ; array prefix = [1,2,3,...]
```

**Flow:**

1. no
2. yes
3. == T
4. < T (too small)
5. T (too big)
6. left=0, right=n-1
7. left < right?
8. return not found
9. sum = a[left] + a[right]
10. sum vs T
11. return left, right
12. left++ discard smallest
13. right-- discard largest

