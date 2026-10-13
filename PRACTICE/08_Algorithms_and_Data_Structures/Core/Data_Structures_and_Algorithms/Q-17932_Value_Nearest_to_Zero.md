---
type: "practice"
difficulty: "Other"
---

# Value Nearest to Zero

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an integer array readings containing n measured deviations from a baseline of zero. Some deviations are negative (below the baseline) and some are positive (above it).

Return the single value in readings whose magnitude is closest to 0. If two different values are equally close to zero (for example a value and its negation, such as -3 and 3), return the larger of the two values.

The distance of a value v from zero is its absolute value |v|.

Input

readings: an array of n integers.

Output

A single integer: the element closest to zero, breaking ties in favor of the larger value.

#### Cons
traints

1 <= n <= 1000
-10^5 <= readings[i] <= 10^5

#### Example
s & test cases
TEXT
Input: readings = [-4, -2, 1, 4, 8] -> Output: 1
Why: The magnitudes are 4, 2, 1, 4, 8; the smallest magnitude is 1, achieved only by the value 1.

Input: readings = [2, -1, 1] -> Output: 1
Why: Both -1 and 1 have magnitude 1 (the minimum). They tie, so the larger value 1 is returned.

Input: readings = [7] -> Output: 7
Why: A single-element array; the only value is the answer.

Input: readings = [-5, -3, -9] -> Output: -3
Why: All values are negative; magnitudes are 5, 3, 9. The smallest magnitude is 3, so -3 is returned (no positive tie exists).

Input: readings = [0, -2, 3] -> Output: 0
Why: Zero has magnitude 0, the smallest possible, so it wins outright.

Input: readings = [-100000, 100000] -> Output: 100000
Why: Both extremes share magnitude 100000 and tie; the larger value 100000 is returned.

#### Approach

Linear scan with a running best. Keep the element seen so far whose absolute value is smallest. For each new value, replace the best if it has a strictly smaller magnitude, or an equal magnitude but a larger signed value (the tie-breaking rule that prefers the positive of a +/- pair). One O(1) comparison per element makes this optimal; the array need not be sorted.

Complexity

Time: O(n) — a single pass over the array, doing O(1) work per element. Space: O(1) — only a running "best" value is stored, independent of input size.

Python
PYTHON
from typing import List

class Solution:
    def valueNearestZero(self, readings: List[int]) -> int:
        best = readings[0]
        for v in readings:
            if abs(v) < abs(best) or (abs(v) == abs(best) and v > best):
                best = v
        return best
JavaScript
JAVASCRIPT
function valueNearestZero(readings) {
    let best = readings[0];
    for (const v of readings) {
        if (Math.abs(v) < Math.abs(best) || (Math.abs(v) === Math.abs(best) && v > best)) {
            best = v;
        }
    }
    return best;
}
Java
JAVA
class Solution {
    public int valueNearestZero(int[] readings) {
        int best = readings[0];
        for (int v : readings) {
            if (Math.abs(v) < Math.abs(best) || (Math.abs(v) == Math.abs(best) && v > best)) {
                best = v;
            }
        }
        return best;
    }
}
C++
CPP
#include <vector>
#include <cstdlib>
using namespace std;

class Solution {
public:
    int valueNearestZero(vector<int>& readings) {
        int best = readings[0];
        for (int v : readings) {
            if (abs(v) < abs(best) || (abs(v) == abs(best) && v > best)) {
                best = v;
            }
        }
        return best;
    }
};
C
C
#include <stdlib.h>

int valueNearestZero(int* readings, int n) {
    int best = readings[0];
    for (int i = 1; i < n; ++i) {
        int v = readings[i];
        if (abs(v) < abs(best) || (abs(v) == abs(best) && v > best)) {
            best = v;
        }
    }
    return best;
}

#### Follow-up
questions
What if ties should instead resolve to the smaller (more negative) value, or to the first occurrence?
How would you return the index of the closest value rather than the value itself?
If the array were sorted, could you find the answer in O(log n) using binary search around zero?
Extend to return the k values closest to zero.

