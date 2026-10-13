---
type: "practice"
difficulty: "Other"
---

# Minimum Ticks to Stamp All Required Tokens

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Greedy](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Greedy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A stamping press produces small metal tokens in exactly three distinct shapes: rounds, squares, and stars. The press has two independent dies, so on every tick of the machine you may do exactly one of the following:

Stamp two tokens of two different shapes (one token from each of two distinct shapes), or
Stamp one token of a single shape.

You are given an integer array need of length 3, where need[0], need[1], and need[2] are the number of round, square, and star tokens you must produce, respectively. You may stamp shapes in any order and are never required to use both dies on a tick.

Return the minimum number of ticks required to produce every token you need.

Input

need: an array of exactly three non-negative integers.

Output

A single integer: the least number of ticks needed to stamp all required tokens.

#### Cons
traints

need.length == 3
0 <= need[i] <= 100 for each i

#### Example
s & test cases
TEXT
Input: need = [1,4,2] -> Output: 4
Why: Pair a round with a square, then square+star, square+star, and finally one lone square. Four ticks is optimal; the square count (4) dominates but total pairing can still absorb the rest.

Input: need = [5,4,4] -> Output: 7
Why: Total tokens = 13, and no single shape exceeds the sum of the other two, so pairing two shapes every tick gives ceil(13/2) = 7.

Input: need = [5,0,0] -> Output: 5
Why: Only one shape has tokens, so no pairing is ever possible; each of the 5 rounds needs its own tick.

Input: need = [0,0,0] -> Output: 0
Why: Nothing to stamp, so zero ticks.

Input: need = [100,1,1] -> Output: 100
Why: The largest shape (100) far exceeds the other two combined (2); every other shape can piggy-back on a round, but the 100 rounds still bound the answer.

Input: need = [3,3,3] -> Output: 5
Why: Total = 9, max shape (3) is not more than the other two (6), so answer is ceil(9/2) = 5.

#### Approach

This is a greedy counting argument on three quantities. Two facts bound the answer. First, the busiest shape must be stamped one token per tick at best (a shape can be paired with at most one other shape per tick), so the answer is at least the maximum count mx. Second, each tick removes at most two tokens, so the answer is at least ceil(total / 2). Both bounds are simultaneously achievable: if one shape exceeds the sum of the other two, you pair the smaller shapes against it until they run out and then finish the surplus alone, giving mx; otherwise the three counts are balanced enough that you can always pair two shapes each tick, giving ceil(total / 2). Hence the minimum is exactly max(mx, ceil(total / 2)). Note: a naive "sum of the two largest, adjust by halves" formula is incorrect — it overcounts balanced cases — so the two-bound maximum is the reliable closed form.

Complexity

Time: O(1) — a fixed number of additions and comparisons over three elements. Space: O(1) — only scalar accumulators are used, independent of the input values.

Python
PYTHON
from math import ceil
from typing import List

def min_ticks(need: List[int]) -> int:
    total = sum(need)
    mx = max(need)
    half = (total + 1) // 2          # ceil(total / 2)
    return max(mx, half)
JavaScript
JAVASCRIPT
function minTicks(need) {

#### cons
t total = need[0] + need[1] + need[2];

#### cons
t mx = Math.max(need[0], need[1], need[2]);

#### cons
t half = Math.ceil(total / 2);
    return Math.max(mx, half);
}
Java
JAVA
class Solution {
    public int minTicks(int[] need) {
        int total = need[0] + need[1] + need[2];
        int mx = Math.max(need[0], Math.max(need[1], need[2]));
        int half = (total + 1) / 2;        // ceil(total / 2)
        return Math.max(mx, half);
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

int minTicks(vector<int>& need) {
    int total = need[0] + need[1] + need[2];
    int mx = max({need[0], need[1], need[2]});
    int half = (total + 1) / 2;            // ceil(total / 2)
    return max(mx, half);
}
C
C
#include <stdio.h>

int minTicks(int need[3]) {
    int total = need[0] + need[1] + need[2];
    int mx = need[0];
    if (need[1] > mx) mx = need[1];
    if (need[2] > mx) mx = need[2];
    int half = (total + 1) / 2;   /* ceil(total / 2) */
    return mx > half ? mx : half;
}

int main(void) {
    int a[3] = {1, 4, 2};
    printf("%d\n", minTicks(a));  /* 4 */
    return 0;
}

#### Follow-up
questions
Generalize to k shapes where each tick may stamp up to m tokens of distinct shapes; does the max(largest, ceil(total/m)) pattern still hold?
Reconstruct one optimal stamping schedule (the actual pairings per tick), not just the count.
Suppose the two dies must always stamp two different shapes (single-token ticks forbidden); when is the task impossible, and what is the minimum then?
If values can be as large as 10^9, confirm the formula still runs in O(1) without overflow and adapt the integer types.

