---
type: "practice"
difficulty: "Other"
---

# Largest Axis-Aligned Rectangle Under a Bar Skyline

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Monotonic Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Monotonic_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array bars of non-negative integers describing a skyline built from unit-width columns standing side by side on a flat baseline. bars[i] is the height of the column at position i, and every column has width exactly 1. Columns touch each other with no gaps, so together they form a solid stair-step silhouette.

#### Cons
ider every axis-aligned rectangle that can be drawn so that it lies entirely inside this silhouette (the rectangle sits on the baseline or above it and never pokes out above any column it spans). A rectangle that spans a contiguous block of columns from position i to position j is limited in height by the shortest column in that block, and its width is the number of columns it covers, j - i + 1. Its area is height * width.

Return the maximum possible area of any such rectangle.

Input

bars: an array of n non-negative integers (the column heights).

Output

A single integer: the largest rectangular area that fits under the skyline.

#### Cons
traints

1 <= n <= 10^5
0 <= bars[i] <= 10^4
The answer fits comfortably in a 64-bit integer (worst case 10^4 * 10^5 = 10^9, which also exceeds nothing beyond a 32-bit signed range only slightly — use a 64-bit accumulator to be safe).

#### Example
s & test cases
TEXT
Input: bars = [2, 1, 5, 6, 2, 3] -> Output: 10
Why: the two columns of heights 5 and 6 (positions 2..3) support a rectangle of height 5 over width 2? No — the best is height 5 across positions 2..3 giving 10; spanning wider forces height down to 2 or 1, which is smaller.

Input: bars = [2, 4] -> Output: 4
Why: taking just the height-4 column gives 4*1 = 4, which beats spanning both columns at the limiting height 2 (2*2 = 4, a tie) — the maximum is 4.

Input: bars = [6] -> Output: 6
Why: a single column is itself a 1-wide rectangle of area 6.

Input: bars = [0, 0, 0] -> Output: 0
Why: every column has height 0, so no rectangle with positive area fits.

Input: bars = [3, 3, 3, 3] -> Output: 12
Why: all equal heights let a single rectangle span the full width 4 at height 3, area 12.

Input: bars = [4, 2, 0, 3, 2, 5] -> Output: 6
Why: the zero at position 2 splits the skyline; the right block [3,2,5] yields height 2 over width 3 = 6, which beats the left block's best (2*2 = 4).

#### Approach

Monotonic (increasing) stack. For each column we want the widest span in which it is the shortest bar; that span, times the column's own height, is the largest rectangle capped by that column, and the overall answer is the max of these. Walk left to right keeping a stack of indices whose heights are strictly increasing. When the current column is shorter than the height on top of the stack, that popped column can no longer extend rightward — so we finalize it now: its height is bars[popped], and its width stretches from just after the new top of the stack up to just before the current index (i - stack.top - 1, or i if the stack is empty). A sentinel of height 0 processed at index n guarantees every remaining bar is popped and measured. Each index enters and leaves the stack once, giving linear time.

Complexity

Time: O(n) — each index is pushed onto the stack exactly once and popped at most once, so the total number of stack operations is linear despite the nested while loop. Space: O(n) — in the worst case (strictly increasing heights) every index sits on the stack simultaneously.

Python
PYTHON
from typing import List

def largest_skyline_rectangle(bars: List[int]) -> int:
    best = 0
    stack = []                       # indices, strictly increasing heights
    n = len(bars)
    for i in range(n + 1):
        # A virtual height of 0 at index n flushes everything on the stack.
        while stack and (i == n or bars[stack[-1]] > bars[i]):
            h = bars[stack.pop()]
            w = i - stack[-1] - 1 if stack else i
            best = max(best, h * w)
        stack.append(i)
    return best
JavaScript
JAVASCRIPT
function largestSkylineRectangle(bars) {
    let best = 0;

#### cons
t stack = [];                // indices, increasing heights

#### cons
t n = bars.length;
    for (let i = 0; i <= n; i++) {
        while (stack.length > 0 && (i === n || bars[stack[stack.length - 1]] > bars[i])) {

#### cons
t h = bars[stack.pop()];

#### cons
t w = stack.length === 0 ? i : i - stack[stack.length - 1] - 1;
            best = Math.max(best, h * w);
        }
        stack.push(i);
    }
    return best;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int largestSkylineRectangle(int[] bars) {
        long best = 0;
        Deque<Integer> stack = new ArrayDeque<>();   // indices, increasing heights
        int n = bars.length;
        for (int i = 0; i <= n; i++) {
            while (!stack.isEmpty() && (i == n || bars[stack.peek()] > bars[i])) {
                int h = bars[stack.pop()];
                int w = stack.isEmpty() ? i : i - stack.peek() - 1;
                best = Math.max(best, (long) h * w);
            }
            stack.push(i);
        }
        return (int) best;
    }
}
C++
CPP
#include <vector>
#include <stack>
#include <algorithm>
using namespace std;

class Solution {
public:
    int largestSkylineRectangle(vector<int>& bars) {
        long best = 0;
        stack<int> st;                       // indices, increasing heights
        int n = static_cast<int>(bars.size());
        for (int i = 0; i <= n; ++i) {
            while (!st.empty() && (i == n || bars[st.top()] > bars[i])) {
                int h = bars[st.top()];
                st.pop();
                int w = st.empty() ? i : i - st.top() - 1;
                best = max(best, static_cast<long>(h) * w);
            }
            st.push(i);
        }
        return static_cast<int>(best);
    }
};
C
C
#include <stdlib.h>

int largestSkylineRectangle(int* bars, int barsSize) {
    /* Monotonic stack of indices with strictly increasing heights. */
    int* stack = (int*)malloc(sizeof(int) * (barsSize + 1));
    int top = -1;
    long best = 0;
    for (int i = 0; i <= barsSize; i++) {
        /* A virtual height of 0 at index barsSize flushes the stack. */
        while (top >= 0 && (i == barsSize || bars[stack[top]] > bars[i])) {
            int h = bars[stack[top--]];
            int w = (top < 0) ? i : i - stack[top] - 1;
            long area = (long)h * w;
            if (area > best) best = area;
        }
        stack[++top] = i;
    }
    free(stack);
    return (int)best;
}

#### Follow-up
questions
Extend to a 2D binary grid: find the largest all-ones rectangle by treating each row as a histogram baseline and reusing this routine.
Return the actual left/right boundaries and height of the winning rectangle, not just its area.
Solve it with a divide-and-conquer approach and compare its worst-case complexity to the stack method.
How does the algorithm behave with many equal heights, and why does using strict > versus >= when popping still produce the correct maximum?

