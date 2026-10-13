---
type: "practice"
difficulty: "Other"
---

# Shortest Contiguous Block Whose Bitwise OR Meets a Threshold

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sliding Window](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array values of non-negative integers and a non-negative integer target.

For any contiguous, non-empty block (subarray) of values, define its strength as the bitwise OR of all elements inside that block. A block is called qualifying if its strength is at least target (that is, strength >= target).

Return the length of the shortest qualifying block. If no contiguous block has strength at least target, return -1.

Note that a block must consist of consecutive elements of values and must contain at least one element. When target is 0, every non-empty block qualifies (its OR is always at least 0), so the answer is 1.

#### Cons
traints:

1 <= values.length <= 2 * 10^5
0 <= values[i] <= 10^9
0 <= target <= 10^9

#### Example
s & test cases
TEXT
Input: values = [1,2,3], target = 2 -> Output: 1
Why: The single-element block [3] has OR = 3 >= 2, so a block of length 1 already qualifies.

Input: values = [2,1,8], target = 10 -> Output: 3
Why: No shorter block reaches 10; the whole array has OR = 2|1|8 = 11 >= 10, giving length 3.

Input: values = [1,2], target = 0 -> Output: 1
Why: target is 0, so any single element qualifies (OR >= 0 is always true).

Input: values = [4,4,4], target = 8 -> Output: -1
Why: Every element is 4 and OR of any block is 4 < 8; no block ever reaches the threshold.

Input: values = [1,0,0,0,8], target = 9 -> Output: 5
Why: Bit 0 comes only from index 0 and bit 3 only from index 4, so the OR reaches 9 only when the block spans the entire array.

Input: values = [7], target = 5 -> Output: 1
Why: The lone element 7 already has OR = 7 >= 5, so the minimal length is 1.

#### Approach

Bitwise OR is monotonic under extension but not directly reversible, so we cannot subtract an element from an OR. Instead, maintain a per-bit counter cnt[b] = how many elements currently inside the sliding window have bit b set. The window's OR is then reconstructed as the set of bits whose counter is positive. Slide right forward adding each new element's bits; whenever the current OR is at least target, record the window length and advance left, decrementing the counters of the element leaving the window. Because counters make removal reversible in O(32), this two-pointer (sliding window) scan finds the shortest qualifying window in linear time. Values are below 2^30, so 32 bit-positions cover every possible bit safely.

Complexity

Time: O(32 * n) = O(n) — each of the two pointers moves forward at most n times, and every add/remove/recompute touches a fixed 32 bits. Space: O(1) — a constant array of 32 bit-counters.

Python
PYTHON
from typing import List

def shortest_qualifying_block(values: List[int], target: int) -> int:
    cnt = [0] * 32                       # cnt[b] = window elements with bit b set
    res = float('inf')
    left = 0

    for right, x in enumerate(values):
        for b in range(32):
            if (x >> b) & 1:
                cnt[b] += 1

        while left <= right:
            cur = 0
            for b in range(32):
                if cnt[b]:
                    cur |= (1 << b)
            if cur < target:
                break

            res = min(res, right - left + 1)

            y = values[left]
            for b in range(32):
                if (y >> b) & 1:
                    cnt[b] -= 1
            left += 1

    return -1 if res == float('inf') else res
JavaScript
JS
function shortestQualifyingBlock(values, target) {

#### cons
t cnt = new Array(32).fill(0);   // cnt[b] = window elements with bit b set
    let res = Infinity, left = 0;

    for (let right = 0; right < values.length; right++) {
        for (let b = 0; b < 32; b++)
            if ((values[right] >> b) & 1) cnt[b]++;

        while (left <= right) {
            let cur = 0;
            for (let b = 0; b < 32; b++)
                if (cnt[b]) cur |= (1 << b);
            if (cur < target) break;

            res = Math.min(res, right - left + 1);

            for (let b = 0; b < 32; b++)
                if ((values[left] >> b) & 1) cnt[b]--;
            left++;
        }
    }
    return res === Infinity ? -1 : res;
}
Java
JAVA
class Solution {
    public int shortestQualifyingBlock(int[] values, int target) {
        int[] cnt = new int[32];
        int res = Integer.MAX_VALUE, left = 0, n = values.length;

        for (int right = 0; right < n; right++) {
            for (int b = 0; b < 32; b++)
                if (((values[right] >> b) & 1) == 1) cnt[b]++;

            while (left <= right) {
                long cur = 0;
                for (int b = 0; b < 32; b++)
                    if (cnt[b] > 0) cur |= (1L << b);
                if (cur < target) break;

                res = Math.min(res, right - left + 1);

                for (int b = 0; b < 32; b++)
                    if (((values[left] >> b) & 1) == 1) cnt[b]--;
                left++;
            }
        }
        return res == Integer.MAX_VALUE ? -1 : res;
    }
}
C++
CPP
#include <vector>
#include <climits>
#include <algorithm>
using namespace std;

class Solution {
public:
    int shortestQualifyingBlock(vector<int>& values, int target) {
        int cnt[32] = {0};
        int res = INT_MAX, left = 0, n = (int)values.size();

        for (int right = 0; right < n; right++) {
            for (int b = 0; b < 32; b++)
                if ((values[right] >> b) & 1) cnt[b]++;

            while (left <= right) {
                long cur = 0;
                for (int b = 0; b < 32; b++)
                    if (cnt[b]) cur |= (1L << b);
                if (cur < (long)target) break;

                res = min(res, right - left + 1);

                for (int b = 0; b < 32; b++)
                    if ((values[left] >> b) & 1) cnt[b]--;
                left++;
            }
        }
        return res == INT_MAX ? -1 : res;
    }
};
C
C
#include <limits.h>

int shortestQualifyingBlock(int* values, int valuesSize, int target) {
    int cnt[32] = {0};                 // cnt[b] = how many window elements have bit b set
    int res = INT_MAX, left = 0;

    for (int right = 0; right < valuesSize; right++) {
        for (int b = 0; b < 32; b++)
            if ((values[right] >> b) & 1) cnt[b]++;

        // shrink from the left while the window's OR still meets the target
        while (left <= right) {
            long cur = 0;
            for (int b = 0; b < 32; b++)
                if (cnt[b]) cur |= (1L << b);
            if (cur < (long)target) break;

            int len = right - left + 1;
            if (len < res) res = len;

            for (int b = 0; b < 32; b++)
                if ((values[left] >> b) & 1) cnt[b]--;
            left++;
        }
    }
    return res == INT_MAX ? -1 : res;
}

#### Follow-up
questions
How would you adapt the counter trick if the operation were bitwise AND instead of OR?
Can you find the shortest block whose OR is exactly equal to target, not just at least target?
What changes if elements can be as large as 10^18, so 64 bit-positions are needed?
Instead of the shortest block, return the count of all blocks whose OR is at least target.

