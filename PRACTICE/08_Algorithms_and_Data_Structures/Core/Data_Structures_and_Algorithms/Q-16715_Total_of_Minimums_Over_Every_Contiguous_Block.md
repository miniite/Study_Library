---
type: "practice"
difficulty: "Other"
---

# Total of Minimums Over Every Contiguous Block

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Monotonic Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Monotonic_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array of positive integers weights. Consider every contiguous block (subarray) of weights — that is, every run of one or more consecutive elements. For each such block, take its minimum element. Sum these minimums across all possible contiguous blocks.

Because the total can be enormous, return the result modulo 1000000007 (that is, 10^9 + 7).

A contiguous block is defined by a start index i and end index j with 0 <= i <= j < n; it contains the elements weights[i], weights[i+1], ..., weights[j].

Input
weights: an array of positive integers.
Output
A single integer: the sum, over all contiguous blocks, of each block's minimum element, taken modulo 1000000007.
Worked reasoning

For weights = [4, 2, 3, 5], the contiguous blocks and their minimums are:

length 1: [4]->4, [2]->2, [3]->3, [5]->5
length 2: [4,2]->2, [2,3]->2, [3,5]->3
length 3: [4,2,3]->2, [2,3,5]->2
length 4: [4,2,3,5]->2

Adding these gives 4+2+3+5 + 2+2+3 + 2+2 + 2 = 27.

#### Cons
traints
1 <= weights.length <= 3 * 10^4
1 <= weights[i] <= 3 * 10^4

#### Example
s & test cases
TEXT
Input: weights = [4, 2, 3, 5] -> Output: 27
  Why: sum of the minimums of all 10 contiguous blocks, as worked out above.

Input: weights = [7] -> Output: 7
  Why: the only block is [7]; its minimum is 7.

Input: weights = [5, 5, 5] -> Output: 30
  Why: minimums are 5 for each of the 6 blocks ([5]x3, [5,5]x2, [5,5,5]x1) -> 6*5 = 30; verifies duplicate handling is not double-counted.

Input: weights = [12, 82, 95, 44, 4] -> Output: 459
  Why: a mixed case with a strong minimum near the end (4 dominates many blocks); confirms the contribution counting across varied values.

Input: weights = [2, 9, 7, 8, 3, 4, 6, 1] -> Output: 117
  Why: 8-element case with the global minimum (1) at the far right, exercising left/right span computation.

Input: weights = [1, 2, 3, 4, 5] -> Output: 35
  Why: strictly increasing array; each element is the minimum of every block starting at or before it and ending at it.

#### Approach

Instead of enumerating all O(n^2) subarrays, count each element's CONTRIBUTION. An element weights[i] is the minimum of exactly those subarrays whose left endpoint lies strictly after the previous smaller element and whose right endpoint lies strictly before the next smaller element. Let left = i - prev[i] be the number of choices for the start (distance to the nearest element on the left with a value <= weights[i]) and right = next[i] - i be the number of choices for the end (distance to the nearest element on the right with a value < weights[i]). Then weights[i] contributes weights[i] * left * right to the answer. The asymmetric tie-breaking — using <= on one side and < on the other — guarantees each subarray's minimum is attributed to exactly one index, so duplicates are never double-counted. Both boundary arrays are computed in one pass with a monotonic increasing stack. This is the canonical "monotonic stack + contribution/span counting" pattern.

Complexity

Time: O(n) — each index is pushed onto and popped from the monotonic stack at most once, and the contribution pass is a single linear sweep. Space: O(n) for the previous-boundary, next-boundary, and stack arrays.

Python
PYTHON
from typing import List

class Solution:
    def totalOfMinimums(self, weights: List[int]) -> int:
        MOD = 1_000_000_007
        n = len(weights)
        prev = [-1] * n   # nearest left index with value <= weights[i]
        nxt = [n] * n     # nearest right index with value <  weights[i]
        stack = []

        for i, w in enumerate(weights):
            while stack and weights[stack[-1]] > w:
                nxt[stack.pop()] = i
            if stack:
                prev[i] = stack[-1]
            stack.append(i)

        ans = 0
        for i, w in enumerate(weights):
            ans += w * (i - prev[i]) * (nxt[i] - i)
            ans %= MOD
        return ans
JavaScript
JAVASCRIPT
/**
 * @param {number[]} weights
 * @return {number}
 */
var totalOfMinimums = function (weights) {

#### cons
t MOD = 1000000007n;

#### cons
t n = weights.length;

#### cons
t prev = new Array(n).fill(-1); // nearest left index with value <= weights[i]

#### cons
t next = new Array(n).fill(n);  // nearest right index with value <  weights[i]

#### cons
t stack = [];

    for (let i = 0; i < n; i++) {
        while (stack.length && weights[stack[stack.length - 1]] > weights[i]) {
            next[stack.pop()] = i;
        }
        if (stack.length) prev[i] = stack[stack.length - 1];
        stack.push(i);
    }

    let ans = 0n;
    for (let i = 0; i < n; i++) {

#### cons
t contrib =
            BigInt(weights[i]) * BigInt(i - prev[i]) * BigInt(next[i] - i);
        ans = (ans + contrib) % MOD;
    }
    return Number(ans);
};
Java
JAVA
import java.util.ArrayDeque;
import java.util.Arrays;
import java.util.Deque;

class Solution {
    public int totalOfMinimums(int[] weights) {
        final long MOD = 1_000_000_007L;
        final int n = weights.length;
        int[] prev = new int[n]; // nearest left index with value <= weights[i]
        int[] next = new int[n]; // nearest right index with value <  weights[i]
        Arrays.fill(prev, -1);
        Arrays.fill(next, n);
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i < n; ++i) {
            while (!stack.isEmpty() && weights[stack.peek()] > weights[i]) {
                next[stack.pop()] = i;
            }
            if (!stack.isEmpty()) prev[i] = stack.peek();
            stack.push(i);
        }

        long ans = 0;
        for (int i = 0; i < n; ++i) {
            long contrib = (long) weights[i]
                * (i - prev[i]) % MOD
                * (next[i] - i) % MOD;
            ans = (ans + contrib) % MOD;
        }
        return (int) ans;
    }
}
C++
CPP
#include <vector>
#include <stack>
using namespace std;

class Solution {
public:
    int totalOfMinimums(vector<int>& weights) {

#### cons
texpr long long MOD = 1'000'000'007;

#### cons
t int n = weights.size();
        vector<int> prev(n, -1); // nearest left index with value <= weights[i]
        vector<int> next(n, n);  // nearest right index with value <  weights[i]
        stack<int> st;

        for (int i = 0; i < n; ++i) {
            while (!st.empty() && weights[st.top()] > weights[i]) {
                next[st.top()] = i;
                st.pop();
            }
            if (!st.empty()) prev[i] = st.top();
            st.push(i);
        }

        long long ans = 0;
        for (int i = 0; i < n; ++i) {
            long long contrib = (long long)weights[i]
                * (i - prev[i]) % MOD
                * (next[i] - i) % MOD;
            ans = (ans + contrib) % MOD;
        }
        return (int)ans;
    }
};
C
C
#include <stddef.h>

#define MOD 1000000007LL

int totalOfMinimums(int* weights, int n) {
    int* prev = (int*)malloc(sizeof(int) * n); /* nearest left index with value <= weights[i], else -1 */
    int* next = (int*)malloc(sizeof(int) * n); /* nearest right index with value <  weights[i], else n  */
    int* stack = (int*)malloc(sizeof(int) * n);
    int top = -1;

    for (int i = 0; i < n; i++) { prev[i] = -1; next[i] = n; }

    for (int i = 0; i < n; i++) {
        while (top >= 0 && weights[stack[top]] > weights[i]) {
            next[stack[top]] = i;
            top--;
        }
        if (top >= 0) prev[i] = stack[top];
        stack[++top] = i;
    }

    long long ans = 0;
    for (int i = 0; i < n; i++) {
        long long contrib = (long long)weights[i]
            * (i - prev[i]) % MOD
            * (next[i] - i) % MOD;
        ans = (ans + contrib) % MOD;
    }

    free(prev); free(next); free(stack);
    return (int)ans;
}

#### Follow-up
questions
How would you compute the sum of subarray MAXIMUMS instead? What changes in the stack comparison?
Can you compute the sum of (max - min) over all subarrays in a single combined pass?
What if you need the sum of the minimums only for subarrays of length exactly k, or at most k?
Explain precisely why the tie-breaking must be strict on one side and non-strict on the other, with a concrete duplicate example.

