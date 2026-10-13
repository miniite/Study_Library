---
type: "practice"
difficulty: "Other"
---

# Longest Block Beating a Length-Scaled Bound

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Monotonic Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Monotonic_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an integer array strength and an integer limit.

#### Cons
ider any contiguous block (subarray) of strength that has length L. The block is called valid if every element inside it is strictly greater than limit / L — that is, an exact real-number division, not integer division.

Return the length L of any valid block. If several valid blocks of different lengths exist, you may return the length of any one of them. If no valid block exists, return -1.

A block (subarray) is a contiguous, non-empty sequence of elements taken from the array.

An equivalent way to state validity: a block of length L is valid if and only if min(block) * L > limit, since the smallest element of the block is the tightest one against the bound limit / L.

#### Cons
traints:

1 <= strength.length <= 10^5
1 <= strength[i] <= 10^9
1 <= limit <= 10^9

(The deterministic reference implementations below scan positions left to right and return the width of the first qualifying block they find; since any valid length is acceptable, other correct programs may return a different valid length.)

#### Example
s & test cases
TEXT
Input: strength = [2,5,7,5,2], limit = 10 -> Output: 3
Why: The block [5,7,5] (indices 1..3) has length 3 and minimum 5 > 10/3 ≈ 3.33, so all elements clear the bound. It is the largest block in which the value 5 is the minimum.

Input: strength = [8,4,8,4,9], limit = 9 -> Output: 5
Why: The whole array has length 5 and minimum 4 > 9/5 = 1.8, so every element clears the bound; 5 is a valid answer (shorter valid blocks also exist, but any valid length is accepted).

Input: strength = [1,1,1], limit = 100 -> Output: -1
Why: The best any block can do is length 3 with all values 1, needing 1 > 100/3 ≈ 33.3, which fails; no length works (no-solution edge case).

Input: strength = [3,3,3,3,20], limit = 15 -> Output: 1
Why: No multi-element block qualifies (e.g. the four 3's give 3 > 15/4 = 3.75, false), but the single element 20 forms a length-1 block with 20 > 15/1 = 15 (single-element edge case).

Input: strength = [3,4,3], limit = 6 -> Output: 3
Why: The full block [3,4,3] has length 3 and minimum 3 > 6/3 = 2, so it is valid.

#### Approach

Key observation: a block is valid iff min(block) * length > limit. For a fixed element strength[i], the widest block in which strength[i] is the minimum stretches from just after the previous strictly-smaller element to just before the next strictly-smaller element; every element in that window is >= strength[i]. If even this widest window (where strength[i] is the tightest value) satisfies strength[i] * width > limit, it is a valid block, and its width is a valid answer. Conversely, if some valid block exists, then for its minimum element that element's maximal window is at least as wide and also qualifies — so checking each element's maximal window is sufficient. Find the two boundaries with the standard "previous/next strictly smaller" monotonic-stack technique (the same machinery behind Largest Rectangle in a Histogram) and test each element in O(1). (A plain sliding window does not work here, because the bound limit / L changes with the window length.)

Complexity

Time: O(n) — two monotonic-stack passes (each index is pushed and popped at most once) plus a single O(n) scan. Space: O(n) for the previous/next boundary arrays and the stack.

Python
PYTHON
def find_block_size(strength, limit):
    n = len(strength)
    prev = [-1] * n     # previous strictly-smaller element
    nxt = [n] * n       # next strictly-smaller element
    st = []

    for i in range(n):
        while st and strength[st[-1]] >= strength[i]:
            st.pop()
        prev[i] = st[-1] if st else -1
        st.append(i)

    st = []
    for i in range(n - 1, -1, -1):
        while st and strength[st[-1]] >= strength[i]:
            st.pop()
        nxt[i] = st[-1] if st else n
        st.append(i)

    for i in range(n):
        width = nxt[i] - prev[i] - 1        # largest block where strength[i] is the min
        if strength[i] * width > limit:
            return width
    return -1
JavaScript
JAVASCRIPT
function findBlockSize(strength, limit) {

#### cons
t n = strength.length;

#### cons
t prev = new Array(n).fill(-1);  // previous strictly-smaller element

#### cons
t nxt = new Array(n).fill(n);    // next strictly-smaller element

#### cons
t st = [];

    for (let i = 0; i < n; i++) {
        while (st.length && strength[st[st.length - 1]] >= strength[i]) st.pop();
        prev[i] = st.length ? st[st.length - 1] : -1;
        st.push(i);
    }

    st.length = 0;
    for (let i = n - 1; i >= 0; i--) {
        while (st.length && strength[st[st.length - 1]] >= strength[i]) st.pop();
        nxt[i] = st.length ? st[st.length - 1] : n;
        st.push(i);
    }

    for (let i = 0; i < n; i++) {

#### cons
t width = nxt[i] - prev[i] - 1;   // largest block where strength[i] is the min
        if (strength[i] * width > limit) return width;
    }
    return -1;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int findBlockSize(int[] strength, long limit) {
        int n = strength.length;
        int[] prev = new int[n];
        int[] nxt = new int[n];
        Deque<Integer> st = new ArrayDeque<>();

        // previous strictly-smaller element
        for (int i = 0; i < n; i++) {
            while (!st.isEmpty() && strength[st.peek()] >= strength[i]) st.pop();
            prev[i] = st.isEmpty() ? -1 : st.peek();
            st.push(i);
        }
        // next strictly-smaller element
        st.clear();
        for (int i = n - 1; i >= 0; i--) {
            while (!st.isEmpty() && strength[st.peek()] >= strength[i]) st.pop();
            nxt[i] = st.isEmpty() ? n : st.peek();
            st.push(i);
        }

        for (int i = 0; i < n; i++) {
            long width = (long) nxt[i] - prev[i] - 1;   // block where i is the min
            if ((long) strength[i] * width > limit) return (int) width;
        }
        return -1;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    int findBlockSize(vector<int>& strength, long long limit) {
        int n = (int)strength.size();
        vector<int> prev(n, -1), nxt(n, n), st;

        // previous strictly-smaller element
        for (int i = 0; i < n; i++) {
            while (!st.empty() && strength[st.back()] >= strength[i]) st.pop_back();
            prev[i] = st.empty() ? -1 : st.back();
            st.push_back(i);
        }
        // next strictly-smaller element
        st.clear();
        for (int i = n - 1; i >= 0; i--) {
            while (!st.empty() && strength[st.back()] >= strength[i]) st.pop_back();
            nxt[i] = st.empty() ? n : st.back();
            st.push_back(i);
        }

        for (int i = 0; i < n; i++) {
            long long width = (long long)nxt[i] - prev[i] - 1;  // block where i is the min
            if ((long long)strength[i] * width > limit) return (int)width;
        }
        return -1;
    }
};
C
C
#include <stdlib.h>

/* Returns the length of some valid block, or -1 if none exists. */
int findBlockSize(const int *strength, int n, long long limit) {
    int *prev = (int *)malloc(sizeof(int) * n);
    int *nxt  = (int *)malloc(sizeof(int) * n);
    int *st   = (int *)malloc(sizeof(int) * n);
    int top = 0;

    /* previous strictly-smaller element for each index */
    for (int i = 0; i < n; i++) {
        while (top > 0 && strength[st[top - 1]] >= strength[i]) top--;
        prev[i] = top > 0 ? st[top - 1] : -1;
        st[top++] = i;
    }
    /* next strictly-smaller element for each index */
    top = 0;
    for (int i = n - 1; i >= 0; i--) {
        while (top > 0 && strength[st[top - 1]] >= strength[i]) top--;
        nxt[i] = top > 0 ? st[top - 1] : n;
        st[top++] = i;
    }

    int ans = -1;
    for (int i = 0; i < n; i++) {
        long long width = (long long)nxt[i] - prev[i] - 1;   /* block where i is the min */
        if ((long long)strength[i] * width > limit) { ans = (int)width; break; }
    }

    free(prev); free(nxt); free(st);
    return ans;
}

#### Follow-up
questions
Instead of returning any valid length, return the maximum length over all valid blocks.
How does the algorithm change if the requirement is "greater than or equal to" limit / L instead of strictly greater?
Solve it with a Union Find approach: process elements from largest to smallest, unioning adjacent activated positions, and check each component's size against the value that activated it.
Besides the length, also report the starting index of a valid block.

