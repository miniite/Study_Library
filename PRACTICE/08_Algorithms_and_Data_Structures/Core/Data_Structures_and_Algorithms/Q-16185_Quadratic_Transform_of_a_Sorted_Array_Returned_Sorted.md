---
type: "practice"
difficulty: "Other"
---

# Quadratic Transform of a Sorted Array Returned Sorted

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Two Pointers](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Two_Pointers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an integer array values that is already sorted in ascending order, along with three integer coefficients p, q, and r. Apply the quadratic function

CODE
f(x) = p * x^2 + q * x + r

to every element of values, and return the resulting values in ascending sorted order.

For example, transforming the element x = values[i] produces f(values[i]); you must return all of these transformed numbers, sorted from smallest to largest.

Aim for a single pass over the array: the sorted structure of the input plus the shape of a parabola let you produce the answer in linear time without a general sort.

#### Cons
traints:

1 <= values.length <= 200
-100 <= values[i], p, q, r <= 100
values is sorted in ascending order.
The output array has the same length as values.

Follow up: Can you achieve O(n) time?

#### Example
s & test cases
TEXT
Input: values = [-3, -1, 0, 2, 5], p = 2, q = -1, r = 1 -> Output: [1, 4, 7, 22, 46]
Why: f(x) = 2x^2 - x + 1. Values map to f(-3)=22, f(-1)=4, f(0)=1, f(2)=7, f(5)=46; sorted ascending gives [1,4,7,22,46]. Upward parabola (p > 0): the largest transformed values sit at the two ends of the sorted input.

Input: values = [-2, 0, 3], p = -1, q = 0, r = 4 -> Output: [-5, 0, 4]
Why: f(x) = -x^2 + 4. f(-2)=0, f(0)=4, f(3)=-5; sorted gives [-5,0,4]. Downward parabola (p < 0): the smallest transformed values sit at the ends, so the merge fills from the front.

Input: values = [5], p = 1, q = 1, r = 1 -> Output: [31]
Why: Single-element edge case; f(5) = 25 + 5 + 1 = 31.

Input: values = [-2, 1, 4], p = 0, q = 2, r = 1 -> Output: [-3, 3, 9]
Why: p = 0 makes f linear (f(x) = 2x + 1), which is monotonic, so the extreme transformed values are still at the ends and the two-pointer merge stays correct.

Input: values = [-4, -2, 2, 4], p = 1, q = 3, r = 5 -> Output: [3, 9, 15, 33]
Why: f(x) = x^2 + 3x + 5. f(-4)=9, f(-2)=3, f(2)=15, f(4)=33; sorted gives [3,9,15,33]. Includes negatives on both sides of the vertex.

#### Approach

A quadratic f(x) = px^2 + qx + r traces a parabola whose concavity is fixed by the sign of p. Because the input is already sorted by x, the transformed values are "V-shaped" (or inverted-V) around the vertex: as x runs from smallest to largest, f first moves one way, reaches its extreme at the vertex, then moves back. That means the two ends of the sorted array always hold the two most extreme transformed values. Use a two-pointer merge: if p > 0 the parabola opens upward, so the maximum of any remaining sub-range is at one of the two ends — take the larger end and write it into the answer from the back. If p <= 0 (opens downward, or is linear when p = 0), the minimum is at an end — take the smaller end and write from the front. Each step advances one pointer, giving a single O(n) pass with no general sort. The pattern is the two-pointer merge exploiting monotonic/unimodal structure.

Complexity

Time: O(n) — each element is consumed by exactly one pointer move, with O(1) work per step (evaluate the quadratic at the two ends and place the extreme). Space: O(n) for the output array, or O(1) auxiliary beyond it.

Python
PYTHON
from typing import List

class Solution:
    def sortTransformed(self, values: List[int], p: int, q: int, r: int) -> List[int]:
        def f(x: int) -> int:
            return (p * x + q) * x + r

        n = len(values)
        ans = [0] * n
        upward = p > 0
        lo, hi = 0, n - 1
        idx = n - 1 if upward else 0
        while lo <= hi:
            fl, fr = f(values[lo]), f(values[hi])
            if upward:  # largest values live at the ends -> fill from the back
                if fl > fr:
                    ans[idx] = fl
                    lo += 1
                else:
                    ans[idx] = fr
                    hi -= 1
                idx -= 1
            else:  # smallest values live at the ends -> fill from the front
                if fl < fr:
                    ans[idx] = fl
                    lo += 1
                else:
                    ans[idx] = fr
                    hi -= 1
                idx += 1
        return ans
JavaScript
JAVASCRIPT
function sortTransformed(values, p, q, r) {

#### cons
t f = (x) => (p * x + q) * x + r;

#### cons
t n = values.length;

#### cons
t ans = new Array(n);

#### cons
t upward = p > 0;
    let lo = 0, hi = n - 1;
    let idx = upward ? n - 1 : 0;
    while (lo <= hi) {

#### cons
t fl = f(values[lo]);

#### cons
t fr = f(values[hi]);
        if (upward) {           // largest values at the ends -> fill from the back
            if (fl > fr) { ans[idx--] = fl; lo++; }
            else         { ans[idx--] = fr; hi--; }
        } else {                // smallest values at the ends -> fill from the front
            if (fl < fr) { ans[idx++] = fl; lo++; }
            else         { ans[idx++] = fr; hi--; }
        }
    }
    return ans;
}
Java
JAVA
class Solution {
    public int[] sortTransformed(int[] values, int p, int q, int r) {
        int n = values.length;
        int[] ans = new int[n];
        boolean upward = p > 0;
        int lo = 0, hi = n - 1;
        int idx = upward ? n - 1 : 0;
        while (lo <= hi) {
            int fl = f(values[lo], p, q, r);
            int fr = f(values[hi], p, q, r);
            if (upward) {
                if (fl > fr) { ans[idx--] = fl; lo++; }
                else         { ans[idx--] = fr; hi--; }
            } else {
                if (fl < fr) { ans[idx++] = fl; lo++; }
                else         { ans[idx++] = fr; hi--; }
            }
        }
        return ans;
    }

    private int f(int x, int p, int q, int r) {
        return (p * x + q) * x + r;
    }
}
C++
CPP
#include <vector>

class Solution {
public:
    std::vector<int> sortTransformed(std::vector<int>& values, int p, int q, int r) {
        int n = static_cast<int>(values.size());
        std::vector<int> ans(n);
        bool upward = p > 0;
        int lo = 0, hi = n - 1;
        int idx = upward ? n - 1 : 0;
        auto f = [&] (long x) -> long { return (static_cast<long>(p) * x + q) * x + r; };
        while (lo <= hi) {
            long fl = f(values[lo]);
            long fr = f(values[hi]);
            if (upward) {
                if (fl > fr) { ans[idx--] = static_cast<int>(fl); lo++; }
                else         { ans[idx--] = static_cast<int>(fr); hi--; }
            } else {
                if (fl < fr) { ans[idx++] = static_cast<int>(fl); lo++; }
                else         { ans[idx++] = static_cast<int>(fr); hi--; }
            }
        }
        return ans;
    }
};
C
C
#include <stdlib.h>

static long applyQuad(long x, int p, int q, int r) {
    return ((long)p * x + q) * x + r;
}

/* Returns a newly allocated array of length n (caller frees). */
int *sortTransformed(int *values, int n, int p, int q, int r, int *returnSize) {
    int *ans = (int *)malloc(sizeof(int) * n);
    *returnSize = n;
    int upward = p > 0;
    int lo = 0, hi = n - 1;
    int idx = upward ? n - 1 : 0;
    while (lo <= hi) {
        long fl = applyQuad(values[lo], p, q, r);
        long fr = applyQuad(values[hi], p, q, r);
        if (upward) {
            if (fl > fr) { ans[idx--] = (int)fl; lo++; }
            else         { ans[idx--] = (int)fr; hi--; }
        } else {
            if (fl < fr) { ans[idx++] = (int)fl; lo++; }
            else         { ans[idx++] = (int)fr; hi--; }
        }
    }
    return ans;
}

#### Follow-up
questions
Prove why the two extreme transformed values always sit at the two ends of the sorted input, and handle the linear case p = 0 explicitly.
What changes if the input is not pre-sorted — is O(n) still possible, or does it become O(n log n)?
Extend to a piecewise or cubic transform where the extremes are no longer guaranteed at the ends.
Return the result as a stream/generator that yields values in sorted order without materializing the full array.

