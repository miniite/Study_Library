---
type: "practice"
difficulty: "Other"
---

# Best Left Shift for Maximum Reward

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sum](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed integer array a of length n.

A left shift by an amount k (where 0 <= k < n) rearranges the array into

CODE
[a[k], a[k+1], ..., a[n-1], a[0], a[1], ..., a[k-1]]

That is, the first k elements are moved, in order, to the back and the rest slide to the front. A shift by k = 0 leaves the array unchanged.

After a shift is applied, look at the resulting array. Every position p (0-indexed) whose stored value is at most its own position — that is, value <= p — awards one reward point. The total reward for a given shift is the number of positions that satisfy this condition.

Return the shift amount k that produces the largest total reward. If several shift amounts tie for the largest reward, return the smallest such k.

Input

a: an array of n integers, each in the range [0, n-1].

Output

A single integer: the smallest shift amount k achieving the maximum reward.

#### Cons
traints

1 <= n <= 10^5
0 <= a[i] < n for every i

#### Example
s & test cases
TEXT
Input: a = [3, 1, 0, 2] -> Output: 0
Why: Rewards for k = 0..3 are [3, 3, 2, 2]. The best reward 3 first occurs at k = 0, so the smallest such shift is chosen.

Input: a = [1, 2, 0] -> Output: 2
Why: Shifting to [0, 1, 2] makes every value equal to its index, giving the maximum reward 3 at k = 2.

Input: a = [0] -> Output: 0
Why: A single element 0 satisfies 0 <= 0 for the only possible shift k = 0.

Input: a = [0, 0, 0] -> Output: 0
Why: Every value is 0, so all positions always score no matter the shift; every shift ties at reward 3, and the smallest k = 0 wins.

Input: a = [2, 3, 1, 4, 0] -> Output: 3
Why: Rewards for k = 0..4 are [2, 3, 3, 4, 3]; the unique maximum 4 occurs at k = 3.

#### Approach

Testing all n shifts by rebuilding the array is O(n^2) and too slow. The key observation is per-element: when the array is left-shifted by k, the element originally at index i lands at new position (i - k) mod n. It fails to score exactly when that new position is strictly less than its value v = a[i], i.e. when the new position lies in {0, 1, ..., v-1}. Translating that back to k, the set of shifts for which element i fails is a single contiguous circular block of length v, namely k in [(i - v + 1) mod n, i]. So each element contributes +1 failure to a circular interval of shifts. Accumulate all these intervals with a difference array (splitting into two pieces when an interval wraps past the end), take a prefix sum to get the failure count for every shift, and pick the shift with the fewest failures — that is the one with the highest reward. Scanning left to right and only updating on a strict improvement automatically returns the smallest tie-breaking k. The underlying pattern is range-update-via-difference-array (a discrete prefix-sum / line-sweep technique).

Complexity

Time: O(n) — each element updates a constant number of difference-array entries, then a single linear prefix-sum pass finds the answer. Space: O(n) for the difference array. This is optimal because every element must be examined at least once.

Python
PYTHON
from typing import List

class Solution:
    def bestShift(self, a: List[int]) -> int:
        n = len(a)
        fail = [0] * (n + 1)              # difference array over shift k
        for i in range(n):
            v = a[i]
            if v == 0:                    # a value of 0 always scores
                continue
            R = i                         # failing shifts form circular block [L, R]
            L = (i - v + 1) % n
            if L <= R:
                fail[L] += 1
                fail[R + 1] -= 1
            else:                         # interval wraps around the end
                fail[L] += 1
                fail[n] -= 1
                fail[0] += 1
                fail[R + 1] -= 1

        best_k, best_fail, cur = 0, 0, 0
        for k in range(n):
            cur += fail[k]                # number of failing positions at shift k
            if k == 0 or cur < best_fail: # minimize failures -> maximize reward
                best_fail, best_k = cur, k
        return best_k
JavaScript
JAVASCRIPT
/**
 * @param {number[]} a
 * @return {number} smallest left-shift amount maximizing the reward
 */
function bestShift(a) {

#### cons
t n = a.length;

#### cons
t fail = new Array(n + 1).fill(0); // difference array over shift k
    for (let i = 0; i < n; i++) {

#### cons
t v = a[i];
        if (v === 0) continue;             // value 0 always scores

#### cons
t R = i;                       // failing shifts: circular [L, R]

#### cons
t L = ((i - v + 1) % n + n) % n;
        if (L <= R) {
            fail[L]++;
            fail[R + 1]--;
        } else {                           // interval wraps around
            fail[L]++;
            fail[n]--;
            fail[0]++;
            fail[R + 1]--;
        }
    }
    let bestK = 0, bestFail = 0, cur = 0;
    for (let k = 0; k < n; k++) {
        cur += fail[k];                    // failures at this shift
        if (k === 0 || cur < bestFail) {   // fewer failures -> more reward
            bestFail = cur;
            bestK = k;
        }
    }
    return bestK;
}
Java
JAVA
class Solution {
    // Returns the smallest left-shift that yields the maximum reward.
    public int bestShift(int[] a) {
        int n = a.length;
        int[] fail = new int[n + 1];                 // difference array over shift k
        for (int i = 0; i < n; i++) {
            int v = a[i];
            if (v == 0) continue;                    // value 0 never fails
            int R = i;                               // failing shifts: circular [L, R]
            int L = ((i - v + 1) % n + n) % n;
            if (L <= R) {
                fail[L]++;
                fail[R + 1]--;
            } else {                                 // interval wraps around
                fail[L]++;
                fail[n]--;
                fail[0]++;
                fail[R + 1]--;
            }
        }
        int bestK = 0, bestFail = 0, cur = 0;
        for (int k = 0; k < n; k++) {
            cur += fail[k];                          // failures at this shift
            if (k == 0 || cur < bestFail) {          // fewer failures = higher reward
                bestFail = cur;
                bestK = k;
            }
        }
        return bestK;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    int bestShift(vector<int>& a) {
        int n = a.size();
        vector<int> fail(n + 1, 0);              // difference array over shift k
        for (int i = 0; i < n; i++) {
            int v = a[i];
            if (v == 0) continue;                // value 0 always scores
            int R = i;                           // failing shifts: circular [L, R]
            int L = ((i - v + 1) % n + n) % n;
            if (L <= R) {
                fail[L]++;
                fail[R + 1]--;
            } else {                             // wraps past the array end
                fail[L]++;
                fail[n]--;
                fail[0]++;
                fail[R + 1]--;
            }
        }
        int bestK = 0, bestFail = 0, cur = 0;
        for (int k = 0; k < n; k++) {
            cur += fail[k];                      // failures for this shift
            if (k == 0 || cur < bestFail) {      // minimize failures -> maximize reward
                bestFail = cur;
                bestK = k;
            }
        }
        return bestK;
    }
};
C
C
#include <stdlib.h>

/* Returns the smallest left-shift amount that maximizes the reward. */
int bestShift(int* a, int n) {
    int* fail = (int*)calloc(n + 1, sizeof(int)); /* difference array over k */
    for (int i = 0; i < n; i++) {
        int v = a[i];
        if (v == 0) continue;            /* value 0 never fails, contributes nothing */
        int R = i;                       /* failing shifts form a circular block */
        int L = ((i - v + 1) % n + n) % n;
        if (L <= R) {                    /* interval [L, R] does not wrap */
            fail[L] += 1;
            fail[R + 1] -= 1;
        } else {                         /* interval wraps around the end */
            fail[L] += 1;
            fail[n] -= 1;
            fail[0] += 1;
            fail[R + 1] -= 1;
        }
    }
    int bestK = 0, bestFail = 0, cur = 0;
    for (int k = 0; k < n; k++) {
        cur += fail[k];                  /* number of positions that fail at shift k */
        if (k == 0 || cur < bestFail) {  /* fewer failures == higher reward */
            bestFail = cur;
            bestK = k;
        }
    }
    free(fail);
    return bestK;
}

#### Follow-up
questions
What if you also had to report the maximum reward value itself, not just the shift index? (Track n - min_failures.)
How would you handle a right shift instead of a left shift? (Mirror the index mapping.)
Suppose the scoring rule were value < position (strict) instead of value <= position; how does the failing interval change?
Can you prove the answer with the fewest failures is unique in reward yet may tie in index, and why the left-to-right scan yields the smallest index?

