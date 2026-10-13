---
type: "practice"
difficulty: "Basic"
---

# Given two sorted arrays, merge them into one sorted array in O(n+m) time.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Interval Merging](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Interval_Merging.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The question gives two arrays that are already sorted (say ascending) and asks me to combine them into one array that is also sorted, using O(n+m) time, where n and m are the two array lengths. The interviewer is probing whether I recognize the classic two-pointer merge — the same "merge" step that powers merge sort. The trap they want me to avoid is the naive "concatenate then sort" approach, which is O((n+m) log(n+m)) and throws away the fact that the inputs are already sorted.

Clarifying questions a strong candidate asks:

Ascending or descending order? (I'll assume ascending.)
Are duplicates allowed / should they be kept? (I'll keep them — a merge preserves all elements.)
Can I allocate a new output array, or must I merge in place? (I'll produce a new array of size n+m; I'll mention the in-place variant.)
Any constraint on data type / nulls? (Assume comparable numbers.)

Plan: walk two index pointers, one per array; at each step append the smaller of the two current elements and advance that pointer; when one array is exhausted, copy the remainder of the other. Each element is looked at exactly once, so the total work is linear.

From first principles to the solution

Because both arrays are sorted, the smallest not-yet-placed element of the whole result must be at the front of one of the two arrays. So I keep an index i into array a and j into array b, both starting at 0, and a result list out.

At each step I compare a[i] and b[j]:

If a[i] <= b[j], then a[i] is the smallest remaining element overall → push it to out, do i++.
Otherwise b[j] is smaller → push it, do j++.

I repeat while both pointers are still in range. Eventually one array runs out. The remaining elements of the other array are all larger than everything placed so far and are already sorted among themselves, so I just append them wholesale.

Why <= and not <? Using <= when they are equal makes the merge stable — equal elements from a come out before equal elements from b, preserving relative order. This matters when the arrays carry objects with a sort key plus extra data.

Why it is O(n+m): every comparison advances at least one pointer, and each of the n+m elements is copied exactly once. No element is ever revisited, so the running time is linear. Space is O(n+m) for the output (unavoidable if we return a new array). If the output must be in the pointers' own storage, we can do it in O(1) extra space by filling from the back (see the in-place note below).

Dry run

a = [1, 3, 5], b = [2, 3, 6, 8]. Pointers i=0, j=0.

a[0]=1 <= b[0]=2 → out=[1], i=1
a[1]=3 <= b[0]=2? No → out=[1,2], j=1
a[1]=3 <= b[1]=3? Yes (stable) → out=[1,2,3], i=2
a[2]=5 <= b[1]=3? No → out=[1,2,3,3], j=2
a[2]=5 <= b[2]=6? Yes → out=[1,2,3,3,5], i=3 → a exhausted
Append rest of b from j=2: 6,8 → out=[1,2,3,3,5,6,8]

Result is fully sorted and length 3+4=7. Correct.

#### Common mistakes
& follow-ups
Concatenate-then-sort — works but is O((n+m) log(n+m)); the interviewer will point out it ignores the "sorted" precondition.
Off-by-one / forgetting the tail — after the main loop, you MUST drain whichever array still has elements. A missed tail loses data.
Not handling empty arrays — if one input is empty the loop body never runs and the tail-drain copies the other array; the code handles this naturally, but say so.
In-place merge (the classic Leetcode 88 variant): a has size n+m with the last m slots empty and you must merge b into it. Trick: fill from the end, comparing the largest remaining elements and placing them at the back, which avoids overwriting unread values and keeps it O(n+m) time, O(1) extra space.
k sorted arrays follow-up: generalize with a min-heap of the current fronts → O(N log k) where N is total elements — a natural next probe.

What a strong answer emphasizes: recognizing the two-pointer merge, justifying linear time by "each element touched once," using <= for stability, remembering the tail-copy, and being ready to extend to in-place and k-way merges.

**Comparison:**

```
Approach	Time	Extra space	Uses the "sorted" precondition?
Concatenate + sort	O((n+m) log(n+m))	O(n+m)	No
Two-pointer merge (new array)	O(n+m)	O(n+m)	Yes
In-place back-fill (Leetcode 88)	O(n+m)	O(1)	Yes
k sorted arrays via min-heap	O(N log k)	O(k)	Yes
```

**Code:**

```

def merge_sorted(a, b): """Merge two ascending-sorted lists into one. O(n+m) time, O(n+m) space.""" i, j = 0, 0 out = [] while i < len(a) and j < len(b): if a[i] <= b[j]: # <= keeps the merge stable out.append(a[i]); i += 1 else: out.append(b[j]); j += 1 # exactly one of these tails is non-empty out.extend(a[i:]) out.extend(b[j:]) return out

CODE
Copy

```java
// Java equivalent
int[] merge(int[] a, int[] b) {
    int[] out = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length)
        out[k++] = (a[i] <= b[j]) ? a[i++] : b[j++];
    while (i < a.length) out[k++] = a[i++];
    while (j < b.length) out[k++] = b[j++];
    return out;
}
JAVA
Copy
// In-place variant (Leetcode 88): a has room for m extra at the end, fill from the back.
void mergeInPlace(int[] a, int n, int[] b, int m) {
    int i = n - 1, j = m - 1, k = n + m - 1;
    while (j >= 0)
        a[k--] = (i >= 0 && a[i] > b[j]) ? a[i--] : b[j--];
}
```

**Example input / output:**

```text
Input:  a = [1, 3, 5],   b = [2, 3, 6, 8]
Output: [1, 2, 3, 3, 5, 6, 8]

Edge cases:
Input:  a = [],          b = [4, 9]      -> Output: [4, 9]
Input:  a = [2, 2, 2],   b = [2]         -> Output: [2, 2, 2, 2]   (duplicates kept)
Input:  a = [1, 2],      b = []          -> Output: [1, 2]
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. i=0, j=0, out=[]
6. i<n AND j<m?
7. append remaining of a, thenremaining of b
8. return out
9. a[i] <= b[j]?
10. out += a[i]; i++
11. out += b[j]; j++

