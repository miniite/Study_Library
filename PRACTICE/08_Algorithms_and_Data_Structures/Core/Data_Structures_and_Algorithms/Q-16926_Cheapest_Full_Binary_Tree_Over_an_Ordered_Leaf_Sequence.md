---
type: "practice"
difficulty: "Other"
---

# Cheapest Full Binary Tree Over an Ordered Leaf Sequence

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Monotonic Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Monotonic_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a sequence of positive integers w[0], w[1], ..., w[n-1], representing leaf values written from left to right.

#### Cons
ider every full binary tree (a tree in which each node has either 0 or exactly 2 children) whose leaves, read left to right in an in-order traversal, are exactly w[0], w[1], ..., w[n-1] in that same order. In other words, the relative left-to-right order of the leaves is fixed and must match the input; only the tree's internal shape may vary.

Each internal (non-leaf) node is assigned a value equal to the product of the largest leaf value in its left subtree and the largest leaf value in its right subtree. The cost of a tree is the sum of the values of all of its internal nodes.

Return the minimum possible cost over all such trees. The answer is guaranteed to fit in a signed 32-bit integer.

Input

w: an array of n positive integers giving the leaf values in fixed left-to-right order.

Output

The smallest achievable sum of all internal-node values.

#### Cons
traints

2 <= n <= 40
1 <= w[i] <= 15
The final answer is less than 2^31.

#### Example
s & test cases
TEXT
- Input: `w = [4, 9]` -> Output: `36`
  Why: with two leaves there is exactly one tree; its single internal node costs `max(4) * max(9) = 4 * 9 = 36` (smallest-size input).
- Input: `w = [5, 3, 7]` -> Output: `50`
  Why: pairing `3` with `5` first costs `5*3 = 15`, then the root costs `5*7 = 35` (total `50`), which beats pairing `3` with `7` first (`3*7 + 5*7 = 21 + 35 = 56`).
- Input: `w = [2, 4, 6, 2]` -> Output: `44`
  Why: a case with a duplicate value (`2` appears twice) and a non-trivial merge order; the optimal sequence of merges sums to `44`.
- Input: `w = [1, 2, 3, 4, 5]` -> Output: `40`
  Why: a strictly increasing sequence; always merging the current smallest leaf with its smaller neighbour yields `40`.
- Input: `w = [15, 15]` -> Output: `225`
  Why: both leaves are at the maximum value, so the only internal node costs `15 * 15 = 225` (upper-bound edge case).

#### Approach

Think of building the tree bottom-up as repeatedly picking two adjacent leaves, merging them into an internal node whose cost is the product of the two leaves' maxima, and replacing the pair by the larger of the two (since the larger value is what any future merge on that side will use). The key greedy insight: the cheapest way to eliminate a leaf is to multiply it by the smaller of its two neighbours, and the best global strategy is to remove the smallest leaf as early as possible so it is multiplied by a small neighbour rather than getting stuck between two large values later. A monotonic-decreasing stack realizes this: while the incoming value a is at least the stack top, that top is a local minimum bordered by a on the right and the stack element below it on the left, so we pop it and add top * min(left, a). After the sweep, any strictly decreasing chain that remains is merged from the top down, each element multiplied by the next-larger element beneath it. This is the classic monotonic-stack / greedy-elimination pattern; an equivalent but slower interval DP (dp(i,j) = min over split k of dp(i,k)+dp(k+1,j)+max(i..k)*max(k+1..j)) confirms correctness.

Complexity

Time: O(n) — each leaf is pushed once and popped at most once from the monotonic stack, so total work is linear. Space: O(n) — the stack holds at most the leaves plus one sentinel. Justification: the greedy stack replaces the O(n^3) interval DP by processing each element a constant number of amortized times.

Python
PYTHON
class Solution:
    def cheapestLeafMergeTree(self, w: list[int]) -> int:
        st = [float('inf')]          # sentinel keeps the stack non-empty
        res = 0
        for a in w:
            while st[-1] <= a:
                mid = st.pop()       # absorb the smallest local leaf now
                res += mid * min(st[-1], a)
            st.append(a)
        while len(st) > 2:           # collapse the leftover decreasing chain
            top = st.pop()
            res += top * st[-1]
        return res
JavaScript
JAVASCRIPT
function cheapestLeafMergeTree(w) {

#### cons
t st = [Number.MAX_SAFE_INTEGER]; // sentinel
    let res = 0;
    for (const a of w) {
        while (st[st.length - 1] <= a) {

#### cons
t mid = st.pop();          // absorb the smallest local leaf
            res += mid * Math.min(st[st.length - 1], a);
        }
        st.push(a);
    }
    while (st.length > 2) {                // collapse the leftover decreasing chain

#### cons
t top = st.pop();
        res += top * st[st.length - 1];
    }
    return res;
}
Java
JAVA
import java.util.Deque;
import java.util.ArrayDeque;

class Solution {
    public int cheapestLeafMergeTree(int[] w) {
        long res = 0;
        Deque<Integer> st = new ArrayDeque<>();
        st.push(Integer.MAX_VALUE);             // sentinel
        for (int a : w) {
            while (st.peek() <= a) {
                int mid = st.pop();             // absorb the smallest local leaf
                res += (long) mid * Math.min(st.peek(), a);
            }
            st.push(a);
        }
        while (st.size() > 2) {                 // collapse the leftover decreasing chain
            int x = st.pop();
            res += (long) x * st.peek();
        }
        return (int) res;
    }
}
C++
CPP
#include <vector>
#include <climits>
using namespace std;

class Solution {
 public:
  int cheapestLeafMergeTree(vector<int>& w) {
    long long res = 0;
    vector<int> st;
    st.push_back(INT_MAX);                 // sentinel
    for (int a : w) {
      while (st.back() <= a) {
        int mid = st.back();               // absorb the smallest local leaf
        st.pop_back();
        res += (long long)mid * min(st.back(), a);
      }
      st.push_back(a);
    }
    while (st.size() > 2) {                 // collapse the leftover decreasing chain
      int x = st.back();
      st.pop_back();
      res += (long long)x * st.back();
    }
    return (int)res;
  }
};
C
C
#include <limits.h>

/* Monotonic-decreasing stack. w has length n. */
int cheapestLeafMergeTree(int* w, int n) {
    long long res = 0;
    int st[64];          /* n <= 40, plus a sentinel */
    int top = 0;
    st[top++] = INT_MAX; /* sentinel so we never pop past the bottom */
    for (int idx = 0; idx < n; ++idx) {
        int a = w[idx];
        while (st[top - 1] <= a) {
            int mid = st[--top];               /* smallest local leaf: absorb it now */
            int left = st[top - 1];
            res += (long long)mid * (left < a ? left : a);
        }
        st[top++] = a;
    }
    while (top > 2) {                           /* collapse the remaining decreasing chain */
        int x = st[--top];
        res += (long long)x * st[top - 1];
    }
    return (int)res;
}

#### Follow-up
questions
Prove the greedy is optimal: why is removing the globally smallest leaf (times its smaller neighbour) never worse than any other first merge?
Solve the same problem with the O(n^3) interval DP and reconcile its answer with the stack approach.
What if internal-node cost used the sum of subtree maxima instead of the product — does the same greedy still apply?
Reconstruct and output one optimal tree (its structure), not just the minimum cost.

