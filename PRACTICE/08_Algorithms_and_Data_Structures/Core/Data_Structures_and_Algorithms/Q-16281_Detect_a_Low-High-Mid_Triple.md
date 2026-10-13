---
type: "practice"
difficulty: "Other"
---

# Detect a Low-High-Mid Triple

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Monotonic Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Monotonic_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array of n integers values. Determine whether the array contains a low-high-mid triple: three positions p, q, r with p < q < r (they need not be adjacent) such that

CODE
values[p] < values[r] < values[q]

In words, choose three elements in index order where the first element is the smallest of the three, the second element is the largest, and the third element lies strictly between the other two. The three chosen elements form a subsequence (order preserved, gaps allowed).

Return true if at least one such triple exists, and false otherwise.

Input: an array values of n integers.

Output: a boolean — true if some p < q < r satisfies values[p] < values[r] < values[q], else false.

#### Cons
traints:

n == values.length
1 <= n <= 2 * 10^5
-10^9 <= values[i] <= 10^9

#### Example
s & test cases
TEXT
Input: values = [2, 4, 6, 8] -> Output: false
Why: the array is strictly increasing, so there is never a later element that dips below an earlier peak — no valid middle value can appear after the high.

Input: values = [4, 2, 5, 3] -> Output: true
Why: pick positions 1, 2, 3 with values 2, 5, 3 -> 2 < 3 < 5, a valid low-high-mid triple.

Input: values = [-2, 5, 4, 1] -> Output: true
Why: -2 (low) at position 0, 5 (high) at position 1, and 4 (mid) at position 2 give -2 < 4 < 5; several other triples work too.

Input: values = [1, 3, 2] -> Output: true
Why: the minimal case 1 < 2 < 3 with 1 first, 3 as the peak, and 2 in between.

Input: values = [3, 2, 1] -> Output: false
Why: strictly decreasing, so no element after a peak can be both below that peak and above an earlier element.

Input: values = [5] -> Output: false
Why: fewer than three elements cannot form any triple.

#### Approach

Monotonic stack scanned from right to left. Iterate the array backward while maintaining (a) a decreasing stack of candidate "high" values (the middle element values[q]) and (b) mid, the largest value that has already been confirmed to sit strictly below some high to its right — i.e. a valid values[r]. For the current element: if it is strictly less than mid, it can act as the "low" values[p] (with the recorded mid as values[r] and the high that dominated it as values[q]), so a triple exists — return true. Otherwise, pop every stack element smaller than the current value; each popped element was a high paired with the current value as a larger high, and its old value becomes the best mid (it is below the current element, which will be pushed as a taller high). Finally push the current element as a new candidate high. Keeping mid as large as possible makes the values[i] < mid test as permissive as possible while still guaranteeing a real high sits to its right. The pattern is the classic monotonic-stack "next greater / candidate tracking" technique run in reverse.

Complexity

Time: O(n) — a single right-to-left pass; each element is pushed onto the stack once and popped at most once, so the inner while loop does amortized O(1) work per element. Space: O(n) for the monotonic stack in the worst case (e.g. a strictly increasing array).

Python
PYTHON
from typing import List

class Solution:
    def hasLowHighMid(self, values: List[int]) -> bool:
        stack = []                       # max stack of candidate "high" values
        mid = float('-inf')              # best confirmed "mid" (values[r]) so far

        for i in range(len(values) - 1, -1, -1):
            if values[i] < mid:          # values[i] serves as the "low" -> triple exists
                return True
            while stack and stack[-1] < values[i]:
                mid = stack.pop()        # popped element is a valid mid below values[i]
            stack.append(values[i])      # values[i] is a candidate "high"
        return False
JavaScript
JAVASCRIPT
function hasLowHighMid(values) {

#### cons
t stack = [];                        // max stack of candidate "high" values
    let mid = Number.NEGATIVE_INFINITY;      // best confirmed "mid" so far

    for (let i = values.length - 1; i >= 0; --i) {
        if (values[i] < mid) {               // values[i] serves as the "low"
            return true;
        }
        while (stack.length && stack[stack.length - 1] < values[i]) {
            mid = stack.pop();               // popped element is a valid mid below values[i]
        }
        stack.push(values[i]);               // values[i] is a candidate "high"
    }
    return false;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public boolean hasLowHighMid(int[] values) {
        Deque<Integer> stack = new ArrayDeque<>(); // max stack of candidate "high" values
        long mid = Long.MIN_VALUE;                 // best confirmed "mid" so far

        for (int i = values.length - 1; i >= 0; --i) {
            if (values[i] < mid)                   // values[i] serves as the "low"
                return true;
            while (!stack.isEmpty() && stack.peek() < values[i])
                mid = stack.pop();                 // popped element is a valid mid below values[i]
            stack.push(values[i]);                 // values[i] is a candidate "high"
        }
        return false;
    }
}
C++
CPP
#include <vector>
#include <stack>
#include <climits>
using namespace std;

class Solution {
public:
    bool hasLowHighMid(vector<int>& values) {
        stack<int> st;                  // max stack of candidate "high" values
        long long mid = LLONG_MIN;      // best confirmed "mid" (values[r]) so far

        for (int i = (int)values.size() - 1; i >= 0; --i) {
            if (values[i] < mid)        // values[i] serves as the "low"
                return true;
            while (!st.empty() && st.top() < values[i]) {
                mid = st.top();         // popped element is a valid mid below values[i]
                st.pop();
            }
            st.push(values[i]);         // values[i] is a candidate "high"
        }
        return false;
    }
};
C
C
#include <stdbool.h>
#include <stdlib.h>
#include <limits.h>

bool hasLowHighMid(int* values, int valuesSize) {
    int* stack = (int*)malloc(sizeof(int) * (valuesSize > 0 ? valuesSize : 1));
    int top = -1;                       /* max stack of candidate "high" values */
    long long mid = LLONG_MIN;          /* best confirmed "mid" (values[r]) so far */

    bool found = false;
    for (int i = valuesSize - 1; i >= 0; --i) {
        if (values[i] < mid) {          /* values[i] can be the "low" -> triple exists */
            found = true;
            break;
        }
        while (top >= 0 && stack[top] < values[i])
            mid = stack[top--];          /* everything popped is a valid mid below values[i] */
        stack[++top] = values[i];        /* values[i] is a candidate "high" */
    }

    free(stack);
    return found;
}

#### Follow-up
questions
Instead of a boolean, return the actual indices p < q < r (or all such triples).
Solve it left to right using an ordered/multiset structure that finds the largest value strictly below the running maximum-so-far.
Generalize to a strictly-increasing-then-drop pattern of length k, or to values[p] < values[r] < values[q] with additional gap constraints.
What changes if equal values are allowed (values[p] <= values[r] <= values[q])?

