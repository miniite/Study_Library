---
type: "practice"
difficulty: "Other"
---

# Longest Run of Ones After One Allowed Flip

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array bits of length n, where every element is either 0 or 1. You are allowed to change at most one 0 into a 1 (you may also choose to change nothing at all).

After performing at most that single change, report the length of the longest contiguous block that consists entirely of 1s.

Return this length as a single integer.

Input

bits: an array of n integers, each equal to 0 or 1.

Output

A single integer: the maximum possible length of a run of consecutive 1s once you have optionally flipped one 0 to 1.

#### Cons
traints

1 <= n <= 10^5
Each bits[i] is either 0 or 1.
The single flip is optional; if the array is already all 1s, the answer is simply n.

#### Example
s & test cases
TEXT
- Input: `bits = [1,1,0,1,1,1]` -> Output: `6`
  Why: flipping the lone `0` joins the run of two on its left with the run of three on its right, covering the whole array.
- Input: `bits = [1,0,1,0,1]` -> Output: `3`
  Why: flipping either interior `0` merges a length-1 run with an adjacent length-1 run and one more `1`, giving three in a row; both zeros cannot be flipped.
- Input: `bits = [1,1,1,1]` -> Output: `4`
  Why: there is no `0` to flip, so the whole already-solid block of `1`s is the answer (flip nothing).
- Input: `bits = [0,0,0]` -> Output: `1`
  Why: flipping any single `0` produces exactly one `1`; the other zeros stay.
- Input: `bits = [0]` -> Output: `1`
  Why: minimal array; flipping its only element yields a single `1`.
- Input: `bits = [1,0,0,1,1]` -> Output: `3`
  Why: flipping the second `0` (index 2) yields `1,0,1,1,1` with a run of three; flipping the first zero only gives two.

#### Approach

This is a sliding-window (two-pointer) problem. Maintain a window [l, r] that is allowed to contain at most one 0 (the single zero we imagine flipping to 1). Extend the right edge r one element at a time. Whenever a second 0 would enter the window, advance the left edge l until exactly one zero remains, decrementing the zero count as the left pointer passes over a 0. Every valid window is a block of 1s with at most one flipped zero, so its length r - l + 1 is a candidate answer; track the maximum over all windows. The invariant "window holds <= 1 zero" is what encodes the single allowed flip, and because both pointers only move rightward the scan is linear.

Complexity

Time: O(n) — each of the two pointers moves forward at most n times, so every index enters and leaves the window once. Space: O(1) — only a few counters are kept; the input is scanned in place.

Python
PYTHON
from typing import List

class Solution:
    def longestRunWithOneFlip(self, bits: List[int]) -> int:
        ans = 0
        zeros = 0
        l = 0
        for r, b in enumerate(bits):
            if b == 0:
                zeros += 1
            while zeros == 2:            # window may contain at most one zero
                if bits[l] == 0:
                    zeros -= 1
                l += 1
            ans = max(ans, r - l + 1)
        return ans
JavaScript
JAVASCRIPT
function longestRunWithOneFlip(bits) {
    let ans = 0, zeros = 0, l = 0;
    for (let r = 0; r < bits.length; ++r) {
        if (bits[r] === 0) zeros++;
        while (zeros === 2) {            // shrink from the left to one zero
            if (bits[l++] === 0) zeros--;
        }
        ans = Math.max(ans, r - l + 1);
    }
    return ans;
}
Java
JAVA
class Solution {
    public int longestRunWithOneFlip(int[] bits) {
        int ans = 0, zeros = 0;
        for (int l = 0, r = 0; r < bits.length; ++r) {
            if (bits[r] == 0) ++zeros;
            while (zeros == 2)            // keep at most one zero in the window
                if (bits[l++] == 0) --zeros;
            ans = Math.max(ans, r - l + 1);
        }
        return ans;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int longestRunWithOneFlip(vector<int>& bits) {
        int ans = 0, zeros = 0;
        for (int l = 0, r = 0; r < (int)bits.size(); ++r) {
            if (bits[r] == 0) ++zeros;
            while (zeros == 2)            // shrink until only one zero remains
                if (bits[l++] == 0) --zeros;
            ans = max(ans, r - l + 1);
        }
        return ans;
    }
};
C
C
#include <stdio.h>

/* Longest run of 1s achievable after flipping at most one 0. */
int longestRunWithOneFlip(const int* bits, int n) {
    int ans = 0, zeros = 0, l = 0;
    for (int r = 0; r < n; ++r) {
        if (bits[r] == 0) ++zeros;
        while (zeros == 2) {          /* window may hold at most one zero */
            if (bits[l] == 0) --zeros;
            ++l;
        }
        int len = r - l + 1;
        if (len > ans) ans = len;
    }
    return ans;
}

int main(void) {
    int a[] = {1, 1, 0, 1, 1, 1};
    printf("%d\n", longestRunWithOneFlip(a, 6)); /* 6 */
    int b[] = {1, 0, 1, 0, 1};
    printf("%d\n", longestRunWithOneFlip(b, 5)); /* 3 */
    return 0;
}

#### Follow-up
questions
Generalize to flipping at most k zeros: keep a window with at most k zeros and the same shrink logic; answer is the largest such window.
Streaming variant: the bits arrive one at a time and cannot all be stored — track only the current window (or the previous run length plus the current run length across one zero) in O(1) memory.
Return the actual indices [l, r] of an optimal window, not just its length.
What if you must flip exactly one zero (the array is guaranteed to contain at least one)? Cap the window so it always spans one flipped zero.

