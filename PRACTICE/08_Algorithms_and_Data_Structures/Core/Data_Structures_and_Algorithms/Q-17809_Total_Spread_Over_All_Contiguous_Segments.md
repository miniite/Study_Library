---
type: "practice"
difficulty: "Other"
---

# Total Spread Over All Contiguous Segments

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Monotonic Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Monotonic_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an integer array arr. For any contiguous, non-empty segment (subarray) of arr, define its spread as the difference between the largest and the smallest element in that segment.

Sum the spread of every contiguous segment of arr, and return that total.

For an array of length n there are n * (n + 1) / 2 such segments; you must add up the spread of all of them. A segment of a single element has spread 0 (its max equals its min), so single elements contribute nothing.

Input: an integer array arr (values may be negative, zero, or positive, and may repeat).

Output: a single integer — the sum of the spreads of all contiguous segments. The result can be large, so use a 64-bit integer type.

#### Cons
traints:

1 <= arr.length <= 1000
-10^9 <= arr[i] <= 10^9

#### Follow-up
the straightforward approach enumerates all segments in O(n^2). Can you achieve O(n) time?

#### Example
s & test cases
TEXT
Input: arr = [4, 1, 5] -> Output: 11
Why: segment spreads are [4]:0, [1]:0, [5]:0, [4,1]:3, [1,5]:4, [4,1,5]:4; total 0+0+0+3+4+4 = 11.

Input: arr = [8, 3, 6] -> Output: 13
Why: [8,3]:5, [3,6]:3, [8,3,6]:5, all singletons 0; total 5+3+5 = 13.

Input: arr = [3, 3] -> Output: 0
Why: every element is equal, so every segment has spread 0.

Input: arr = [-2, -5, -1] -> Output: 11
Why: [-2,-5]:3, [-5,-1]:4, [-2,-5,-1]:4 (max -1, min -5); total 3+4+4 = 11 — negatives handled naturally.

Input: arr = [4, 1] -> Output: 3
Why: only the full segment [4,1] has nonzero spread, equal to 3.

Input: arr = [9] -> Output: 0
Why: a single element has spread 0, so the total is 0 (minimum-size input).

#### Approach

The spread of a segment is max - min, so summing spreads over all segments equals (sum of the maximum of every segment) minus (sum of the minimum of every segment). Each of those two sums is computed with the contribution technique: instead of iterating over segments, ask, for each element arr[i], in how many segments it is the maximum (respectively the minimum). Element arr[i] is the max of exactly left * right segments, where left is the number of positions the segment can start at (up to the previous element that is greater) and right is the number of positions it can end at (up to the next element that is greater). A monotonic stack finds those nearest greater/smaller boundaries in one left-to-right and one right-to-left pass. To count each segment exactly once when equal values appear, tie-breaking is made strict on one side and non-strict on the other (for minimums the left pass pops on >= and the right pass on >; for maximums the inequalities are mirrored). Summing arr[i] * left * right gives the total of maxima (or minima), and the answer is their difference.

Complexity

Time: O(n) — four linear scans (two per extreme) in which each index is pushed and popped from the monotonic stack at most once, plus a linear contribution sum. Space: O(n) for the span arrays and the stack. This meets the O(n) follow-up; the naive enumerate-all-segments approach is O(n^2).

Python
PYTHON
from typing import List

class Solution:
    def subarraySpreadSum(self, arr: List[int]) -> int:
        n = len(arr)

        def sum_extremes(is_min: bool) -> int:
            left = [0] * n
            right = [0] * n
            stack = []
            for i in range(n):
                while stack and (arr[stack[-1]] >= arr[i] if is_min
                                 else arr[stack[-1]] <= arr[i]):
                    stack.pop()
                left[i] = i - (stack[-1] if stack else -1)
                stack.append(i)
            stack = []
            for i in range(n - 1, -1, -1):
                while stack and (arr[stack[-1]] > arr[i] if is_min
                                 else arr[stack[-1]] < arr[i]):
                    stack.pop()
                right[i] = (stack[-1] if stack else n) - i
                stack.append(i)
            return sum(arr[i] * left[i] * right[i] for i in range(n))

        return sum_extremes(False) - sum_extremes(True)
JavaScript
JAVASCRIPT
function subarraySpreadSum(arr) {

#### cons
t n = arr.length;

    function sumExtremes(isMin) {

#### cons
t left = new Array(n).fill(0);

#### cons
t right = new Array(n).fill(0);
        let stack = [];
        for (let i = 0; i < n; ++i) {
            while (stack.length &&
                (isMin ? arr[stack[stack.length - 1]] >= arr[i]
                       : arr[stack[stack.length - 1]] <= arr[i]))
                stack.pop();
            left[i] = i - (stack.length ? stack[stack.length - 1] : -1);
            stack.push(i);
        }
        stack = [];
        for (let i = n - 1; i >= 0; --i) {
            while (stack.length &&
                (isMin ? arr[stack[stack.length - 1]] > arr[i]
                       : arr[stack[stack.length - 1]] < arr[i]))
                stack.pop();
            right[i] = (stack.length ? stack[stack.length - 1] : n) - i;
            stack.push(i);
        }
        let total = 0n;
        for (let i = 0; i < n; ++i)
            total += BigInt(arr[i]) * BigInt(left[i]) * BigInt(right[i]);
        return total;
    }

    return sumExtremes(false) - sumExtremes(true);
}
Java
JAVA
import java.util.*;

class Solution {
    // Sum over all segments of the segment max (isMin=false) or min (isMin=true).
    private long sumExtremes(int[] arr, boolean isMin) {
        int n = arr.length;
        int[] leftSpan = new int[n], rightSpan = new int[n];
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i < n; ++i) {
            while (!stack.isEmpty() &&
                   (isMin ? arr[stack.peek()] >= arr[i] : arr[stack.peek()] <= arr[i]))
                stack.pop();
            leftSpan[i] = i - (stack.isEmpty() ? -1 : stack.peek());
            stack.push(i);
        }
        stack.clear();
        for (int i = n - 1; i >= 0; --i) {
            while (!stack.isEmpty() &&
                   (isMin ? arr[stack.peek()] > arr[i] : arr[stack.peek()] < arr[i]))
                stack.pop();
            rightSpan[i] = (stack.isEmpty() ? n : stack.peek()) - i;
            stack.push(i);
        }
        long total = 0;
        for (int i = 0; i < n; ++i)
            total += (long) arr[i] * leftSpan[i] * rightSpan[i];
        return total;
    }

    public long subarraySpreadSum(int[] arr) {
        return sumExtremes(arr, false) - sumExtremes(arr, true);
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
    // Sum over all segments of the segment max (isMin=false) or min (isMin=true).
    long long sumExtremes(const vector<int>& arr, bool isMin) {
        int n = arr.size();
        vector<int> stack, leftSpan(n), rightSpan(n);

        for (int i = 0; i < n; ++i) {
            while (!stack.empty() &&
                   (isMin ? arr[stack.back()] >= arr[i] : arr[stack.back()] <= arr[i]))
                stack.pop_back();
            leftSpan[i] = i - (stack.empty() ? -1 : stack.back());
            stack.push_back(i);
        }
        stack.clear();
        for (int i = n - 1; i >= 0; --i) {
            while (!stack.empty() &&
                   (isMin ? arr[stack.back()] > arr[i] : arr[stack.back()] < arr[i]))
                stack.pop_back();
            rightSpan[i] = (stack.empty() ? n : stack.back()) - i;
            stack.push_back(i);
        }
        long long total = 0;
        for (int i = 0; i < n; ++i)
            total += (long long)arr[i] * leftSpan[i] * rightSpan[i];
        return total;
    }
public:
    long long subarraySpreadSum(vector<int>& arr) {
        return sumExtremes(arr, false) - sumExtremes(arr, true);
    }
};
C
C
#include <stdlib.h>

/* Sum, over all segments, of the segment's max (isMin=0) or min (isMin=1). */
static long long sumExtremes(int* arr, int n, int isMin) {
    int* stack     = (int*)malloc(sizeof(int) * n);
    int* leftSpan  = (int*)malloc(sizeof(int) * n);
    int* rightSpan = (int*)malloc(sizeof(int) * n);
    int top;

    top = -1;
    for (int i = 0; i < n; ++i) {
        while (top >= 0 &&
               (isMin ? arr[stack[top]] >= arr[i] : arr[stack[top]] <= arr[i]))
            --top;
        leftSpan[i] = i - (top >= 0 ? stack[top] : -1);
        stack[++top] = i;
    }

    top = -1;
    for (int i = n - 1; i >= 0; --i) {
        while (top >= 0 &&
               (isMin ? arr[stack[top]] > arr[i] : arr[stack[top]] < arr[i]))
            --top;
        rightSpan[i] = (top >= 0 ? stack[top] : n) - i;
        stack[++top] = i;
    }

    long long total = 0;
    for (int i = 0; i < n; ++i)
        total += (long long)arr[i] * leftSpan[i] * rightSpan[i];

    free(stack); free(leftSpan); free(rightSpan);
    return total;
}

long long subarraySpreadSum(int* arr, int n) {
    return sumExtremes(arr, n, 0) - sumExtremes(arr, n, 1);
}

#### Follow-up
questions
Start with the O(n^2) enumeration, then derive the O(n) monotonic-stack solution — explain why the two agree.
Walk through how duplicate values are prevented from being double-counted (the strict/non-strict tie-break).
Return the sum of max + min over all segments, or the sum of (max - min)^2, and adapt the method.
Extend the idea to the sum of spreads over all sub-rectangles of a 2D grid — is the contribution trick still enough?

