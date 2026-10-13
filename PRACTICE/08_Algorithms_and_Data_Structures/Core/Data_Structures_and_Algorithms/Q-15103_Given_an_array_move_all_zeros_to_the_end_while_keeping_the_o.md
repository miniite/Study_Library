---
type: "practice"
difficulty: "Basic"
---

# Given an array, move all zeros to the end while keeping the order of non-zero elements.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array In-place Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array_In-place_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

Given an array like [0, 1, 0, 3, 12], we must move every 0 to the end while keeping the relative order of the non-zero elements unchanged, giving [1, 3, 12, 0, 0]. This is a "basic concept" array question, so the interviewer is checking three things: (1) can you keep the non-zero order stable (not just sort or shuffle), (2) can you do it in place without allocating a second array, and (3) do you reach the clean O(n) time, O(1) space two-pointer solution rather than something quadratic. Clarifying points I'd confirm: modify in place vs. return a new array (I'll do in place, which is the standard ask), only literal 0 counts (not other falsy-looking values), and "order preserved" applies to the non-zeros (the zeros are indistinguishable so their order doesn't matter).

Plan: start from the simplest correct idea, show why a naive approach is wasteful, then derive the optimal two-pointer / write-index technique from first principles, dry-run it, cover edge cases, and state complexity.

From first principles

Simplest correct idea (and why it's not ideal). You could make a new array, copy all non-zero elements first, then fill the rest with zeros. That's O(n) time but O(n) extra space and it's not "in place." Fine to mention as a warm-up, but the interviewer wants in place.

Another naive idea: repeatedly find a zero and shift every element after it left by one (like removing from an array), appending a zero at the end. Each removal is O(n) and you may do it O(n) times → O(n²) worst case (e.g. [0,0,0,...,1]). Correct but slow. This motivates a better approach.

The key insight — a "write pointer" for non-zeros. Keep an index insertPos (start at 0) that marks where the next non-zero should go. Walk a read index across the whole array. Every time you see a non-zero, place it at insertPos and advance insertPos by one. Because you read left to right and write left to right, the non-zeros land in the same relative order — that's what preserves stability. After the read pass finishes, insertPos points just past the last non-zero, so every index from insertPos to the end must be 0 — fill them.

That's the two-phase version. A slightly nicer one-pass swap version does the fill implicitly:

One-pass swap version. Keep insertPos = 0. For each read from 0..n-1: if arr[read] != 0, swap arr[insertPos] and arr[read], then insertPos++. This swaps each non-zero forward into place while pushing zeros back. When insertPos == read the swap is a no-op (element already in position), so it's efficient. Both versions are O(n) time, O(1) space and stable. I'd present the swap version as the final answer because it does everything in a single pass with no separate zero-filling loop.

Why it's stable. Non-zero elements are moved strictly left-to-right and never reordered among themselves; each keeps its original relative position. Zeros are interchangeable, so pushing them to the tail in any order is fine.

Why it's in place. We only ever read and write within the same array plus two integer indices — no auxiliary array, hence O(1) extra space.

Edge cases to call out
Empty array [] → nothing to do, return as is.
All zeros [0,0,0] → insertPos never advances; array unchanged (already correct).
No zeros [1,2,3] → every element swaps with itself; unchanged and correct.
Single element → trivially correct.
Zeros already at the end [1,2,0,0] → the algorithm leaves it as is (no wasted moves in the swap version).
All non-zeros then zeros mixed → handled uniformly.
Complexity
Time: O(n) — one pass over the array; each element examined once (and swapped at most once).
Space: O(1) — only two index variables; fully in place.

A subtle optimization: in the swap version, each non-zero is written at most once and the number of swaps equals the number of non-zeros, so it also minimizes writes compared to naively overwriting everything — worth mentioning if the interviewer asks about minimizing writes (relevant when array elements are expensive to write, e.g. flash memory).

Summary — what a strong answer emphasizes

The clean answer is a two-pointer / write-index scan: keep an insertPos, and for each non-zero element write (or swap) it to insertPos and advance. This is O(n) time, O(1) space, in place, and stable. Reading and writing both left-to-right is exactly what preserves the non-zero order.

#### Common mistakes
(1) shifting the array on every zero → O(n²); (2) sorting or using a comparator → breaks relative order and is O(n log n); (3) filling non-zeros then forgetting to fill the tail with zeros in the two-phase version; (4) allocating a second array when the ask is "in place"; (5) treating other falsy values as zero. Follow-up probes: "move zeros to the front instead" (walk right-to-left with a write pointer at the end), "minimize the number of writes" (the swap version already does), and "what if the array is huge / streamed" (the single forward pass is stream-friendly).

**Comparison:**

```
Approach	Time	Space	In place?	Stable?
Copy non-zeros to new array + pad zeros	O(n)	O(n)	No	Yes
Shift-on-each-zero (array-removal style)	O(n²)	O(1)	Yes	Yes
Sort with comparator	O(n log n)	O(1)/O(n)	Yes	breaks order unless stable sort
Two-pointer write-index / swap (chosen)	O(n)	O(1)	Yes	Yes
```

**Code:**

```

def move_zeros(arr): """In place, O(n) time, O(1) space, order of non-zeros preserved.""" insert_pos = 0 for read in range(len(arr)): if arr[read] != 0: arr[insert_pos], arr[read] = arr[read], arr[insert_pos] # swap forward insert_pos += 1 return arr

Two-phase variant (explicit fill) — same complexity, sometimes clearer:

def move_zeros_two_phase(arr): insert_pos = 0 for x in arr: if x != 0: arr[insert_pos] = x insert_pos += 1 for i in range(insert_pos, len(arr)): arr[i] = 0 return arr

CODE
Copy
```java
// Java — swap version
public void moveZeroes(int[] nums) {
    int insertPos = 0;
    for (int read = 0; read < nums.length; read++) {
        if (nums[read] != 0) {
            int tmp = nums[insertPos];
            nums[insertPos] = nums[read];
            nums[read] = tmp;
            insertPos++;
        }
    }
}
```

**Example input / output:**

```

Input : [0, 1, 0, 3, 12] Output: [1, 3, 12, 0, 0]

Input : [0, 0, 1] Output: [1, 0, 0]

Input : [1, 2, 3] (no zeros) Output: [1, 2, 3]

Input : [] (empty) Output: []

CODE
Copy

Dry-run of the swap version on `[0, 1, 0, 3, 12]` (insert_pos starts 0):

read=0 val=0 -> skip arr=[0,1,0,3,12] ip=0 read=1 val=1 -> swap idx0,idx1 ip->1 arr=[1,0,0,3,12] ip=1 read=2 val=0 -> skip arr=[1,0,0,3,12] ip=1 read=3 val=3 -> swap idx1,idx3 ip->2 arr=[1,3,0,0,12] ip=2 read=4 val=12 -> swap idx2,idx4 ip->3 arr=[1,3,12,0,0] ip=3 result: [1, 3, 12, 0, 0] (non-zero order 1,3,12 preserved)
```

**Flow:**

1. no
2. yes
3. no
4. yes
5. insert_pos = 0
6. read = 0
7. read < n ?
8. done (zeros already at tail)
9. arr[read] != 0 ?
10. read++
11. swap arr[insert_pos], arr[read]
12. insert_pos++

