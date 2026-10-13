---
type: "practice"
difficulty: "Other"
---

# Largest Smaller Arrangement Using a Single Swap

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Two Pointers](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Two_Pointers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array seq of positive integers (values are not guaranteed to be distinct).

#### Cons
ider all arrangements you can obtain from seq by performing exactly one swap — that is, by choosing two positions i and j (i != j) and exchanging seq[i] with seq[j]. Among all such arrangements that are strictly lexicographically smaller than the original seq, return the one that is lexicographically largest.

If no single swap produces an arrangement smaller than seq, return seq unchanged.

An array A is lexicographically smaller than an array B (of the same length) if at the first index where they differ, A has the smaller value.

Input
seq: an array of positive integers.
Output
The array after applying the best single swap described above, or the original array if no improving swap exists.

#### Cons
traints
1 <= seq.length <= 10^4
1 <= seq[i] <= 10^4

#### Example
s & test cases
TEXT
Input: seq = [5,4,2] -> Output: [5,2,4]
Why: Swapping the 4 and the 2 lowers position 1 as little as possible while still producing a smaller array; this is the largest arrangement below the original.

Input: seq = [2,3,8] -> Output: [2,3,8]
Why: The array is strictly increasing, so it is already the smallest arrangement; no single swap can make it smaller.

Input: seq = [2,8,3,5,6] -> Output: [2,6,3,5,8]
Why: Position 1 (value 8) is the first place from the left where a later value is smaller; swapping it with the largest later value strictly less than 8 (which is 6) yields the greatest smaller arrangement.

Input: seq = [5,3,4,4] -> Output: [4,3,5,4]
Why: The best value to bring into position 0 is 4 (largest value below 5 that appears later). When that value is duplicated, we must swap with its **leftmost** occurrence so the remaining tail stays as large as possible.

Input: seq = [9] -> Output: [9]
Why: A single element admits no swap, so the array is returned unchanged.

Input: seq = [1,2,1] -> Output: [1,1,2]
Why: Position 1 (value 2) is followed by a smaller value 1; swapping them gives [1,1,2], the largest arrangement strictly below the original.

#### Approach

To get a strictly smaller arrangement, we want to decrease the value at the leftmost possible position — but only where a decrease is actually available. Scan from the right to find the rightmost index l such that seq[l] > seq[l + 1]; this is the last position where some later element is smaller, and it is exactly where a beneficial swap can lower the array. If no such l exists the array is non-increasing-free (non-decreasing), already the smallest possible arrangement, so return it as is. To keep the result as large as possible while still being smaller, place at position l the largest value among seq[l+1..] that is strictly less than seq[l]. When that target value is duplicated, swap with its leftmost occurrence, which keeps the larger duplicates further right and maximizes the tail. The right-to-left scan while seq[r] >= seq[l] or seq[r] == seq[r-1] finds exactly that index. This is a greedy, next-permutation-style technique.

Complexity

Time: O(n) — two linear scans, one to locate the pivot from the right and one to locate the swap partner. Space: O(1) — the swap is done in place with a constant number of indices.

Python
PYTHON
from typing import List

class Solution:
    def prevPermuteOneSwap(self, seq: List[int]) -> List[int]:
        n = len(seq)
        l = n - 2
        while l >= 0 and seq[l] <= seq[l + 1]:
            l -= 1
        if l < 0:
            return seq  # already the smallest arrangement

        r = n - 1
        # largest value strictly less than seq[l], leftmost if duplicated
        while seq[r] >= seq[l] or seq[r] == seq[r - 1]:
            r -= 1

        seq[l], seq[r] = seq[r], seq[l]
        return seq
JavaScript
JAVASCRIPT
function prevPermuteOneSwap(seq) {

#### cons
t n = seq.length;
    let l = n - 2;
    while (l >= 0 && seq[l] <= seq[l + 1]) {
        --l;
    }
    if (l < 0) {
        return seq; // already the smallest arrangement
    }

    let r = n - 1;
    // largest value strictly less than seq[l], leftmost if duplicated
    while (seq[r] >= seq[l] || seq[r] === seq[r - 1]) {
        --r;
    }

    [seq[l], seq[r]] = [seq[r], seq[l]];
    return seq;
}
Java
JAVA
class Solution {
    public int[] prevPermuteOneSwap(int[] seq) {
        final int n = seq.length;
        int l = n - 2;
        while (l >= 0 && seq[l] <= seq[l + 1])
            --l;
        if (l < 0)
            return seq;            // already the smallest arrangement

        int r = n - 1;
        // largest value strictly less than seq[l], leftmost if duplicated
        while (seq[r] >= seq[l] || seq[r] == seq[r - 1])
            --r;

        int tmp = seq[l];
        seq[l] = seq[r];
        seq[r] = tmp;
        return seq;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> prevPermuteOneSwap(vector<int>& seq) {

#### cons
t int n = seq.size();
        int l = n - 2;
        while (l >= 0 && seq[l] <= seq[l + 1])
            --l;
        if (l < 0)
            return seq;             // already the smallest arrangement

        int r = n - 1;
        // largest value strictly less than seq[l], leftmost if duplicated
        while (seq[r] >= seq[l] || seq[r] == seq[r - 1])
            --r;

        swap(seq[l], seq[r]);
        return seq;
    }
};
C
C
#include <stddef.h>

/* Modifies seq in place and returns the same pointer for convenience. */
int* prevPermuteOneSwap(int* seq, int n) {
    int l = n - 2;
    while (l >= 0 && seq[l] <= seq[l + 1])
        --l;
    if (l < 0)
        return seq;                 /* already the smallest arrangement */

    int r = n - 1;
    /* find the largest value strictly less than seq[l], leftmost if duplicated */
    while (seq[r] >= seq[l] || seq[r] == seq[r - 1])
        --r;

    int tmp = seq[l];
    seq[l] = seq[r];
    seq[r] = tmp;
    return seq;
}

#### Follow-up
questions
How would you instead find the lexicographically smallest arrangement strictly larger than seq with one swap?
Solve the classic next-permutation (any number of adjacent moves, not just a single swap) — how does the strategy change?
Why is picking the leftmost occurrence of the target value essential when duplicates are present? Construct an input that breaks a naive rightmost pick.
Generalize to "largest arrangement strictly smaller than seq using at most k swaps."

