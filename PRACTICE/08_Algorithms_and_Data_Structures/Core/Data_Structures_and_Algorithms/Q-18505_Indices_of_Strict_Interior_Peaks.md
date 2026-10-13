---
type: "practice"
difficulty: "Other"
---

# Indices of Strict Interior Peaks

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed integer array elevations that records the height measured at each step along a straight path. Identify every position that is a strict peak and return a list of those indices, in any order.

A position i is a strict peak when its value is strictly greater than both of its immediate neighbors — that is, elevations[i] > elevations[i - 1] and elevations[i] > elevations[i + 1].

By definition the first and last positions can never be peaks, because each of them is missing one neighbor. If the array contains no strict peak, return an empty list.

Input

elevations: an array of integers.

Output

A list containing the index of every strict peak, in any order (empty if there are none).

#### Cons
traints

3 <= elevations.length <= 10^5
1 <= elevations[i] <= 10^9

#### Example
s & test cases
TEXT
- Input: `elevations = [3,5,5]` -> Output: `[]`
  Why: index 1 (value 5) is not strictly greater than its right neighbor (also 5); the two ends are excluded, so there are no peaks.
- Input: `elevations = [2,7,4,9,6]` -> Output: `[1,3]`
  Why: index 1 (`7 > 2` and `7 > 4`) and index 3 (`9 > 4` and `9 > 6`) are strict peaks; index 2 (value 4) is smaller than its neighbors.
- Input: `elevations = [1,2,3]` -> Output: `[]`
  Why: the values only increase, so index 1 is not greater than index 2, and the ends can never be peaks.
- Input: `elevations = [10,1,10]` -> Output: `[]`
  Why: the only interior index (value 1) is a valley, not a peak; the ends are excluded (minimum-length case).
- Input: `elevations = [1,9,1,9,1,9,1]` -> Output: `[1,3,5]`
  Why: in this alternating pattern every interior high point strictly exceeds both of its neighbors.
- Input: `elevations = [5,5,5,6,5]` -> Output: `[3]`
  Why: index 3 (value 6) is the only element strictly greater than both neighbors; the flat run of 5s produces no strict peak (duplicates edge case).

#### Approach

This is a straightforward enumeration / linear-scan pattern. A strict peak is a purely local property — an element that is strictly greater than the element on its left and the element on its right — so we simply walk every interior index i in the range [1, n-2] and test the two comparisons elevations[i] > elevations[i-1] and elevations[i] > elevations[i+1]. Indices that satisfy both are collected into the answer. The first and last indices are skipped up front because they lack one neighbor and therefore can never be peaks. There is no ordering requirement on the output, so the natural left-to-right collection order is acceptable.

Complexity

Time: O(n) — a single left-to-right pass that inspects each interior index once and does O(1) work per index. Space: O(1) auxiliary beyond the output list (only a couple of counters/iterators). Justification: whether an index is a strict peak depends solely on it and its two immediate neighbors, so no sorting or extra data structure is needed.

Python
PYTHON
from typing import List

class Solution:
    def findStrictPeaks(self, elevations: List[int]) -> List[int]:
        # An interior index is a peak iff it strictly exceeds both neighbors.
        return [i for i in range(1, len(elevations) - 1)
                if elevations[i] > elevations[i - 1]
                and elevations[i] > elevations[i + 1]]
JavaScript
JAVASCRIPT
function findStrictPeaks(elevations) {

#### cons
t result = [];
    for (let i = 1; i + 1 < elevations.length; ++i) {          // skip first and last
        if (elevations[i] > elevations[i - 1] &&
            elevations[i] > elevations[i + 1]) {
            result.push(i);
        }
    }
    return result;
}
Java
JAVA
class Solution {
  public List<Integer> findStrictPeaks(int[] elevations) {
    List<Integer> result = new ArrayList<>();
    for (int i = 1; i + 1 < elevations.length; ++i)            // ends can never be peaks
      if (elevations[i] > elevations[i - 1] &&
          elevations[i] > elevations[i + 1])
        result.add(i);
    return result;
  }
}
C++
CPP
class Solution {
 public:
  vector<int> findStrictPeaks(vector<int>& elevations) {
    vector<int> result;
    for (int i = 1; i + 1 < (int)elevations.size(); ++i)       // interior indices only
      if (elevations[i] > elevations[i - 1] &&
          elevations[i] > elevations[i + 1])
        result.push_back(i);
    return result;
  }
};
C
C
/* Returns a newly malloc'd array of peak indices; *returnSize is its length. */
int* findStrictPeaks(int* elevations, int elevationsSize, int* returnSize) {
    int* result = (int*)malloc(sizeof(int) * (elevationsSize > 0 ? elevationsSize : 1));
    int count = 0;
    for (int i = 1; i + 1 < elevationsSize; ++i) {           /* skip the two ends */
        if (elevations[i] > elevations[i - 1] &&
            elevations[i] > elevations[i + 1]) {
            result[count++] = i;
        }
    }
    *returnSize = count;
    return result;
}

#### Follow-up
questions
How does the definition change if plateaus should count, i.e. a peak is a flat top that is >= its neighbors and strictly greater than the values just outside the flat run?
In the same single pass, also return the count of peaks and the maximum peak value.
Extend the idea to a 2D grid, where a cell is a peak if it strictly exceeds its up/down/left/right neighbors.
If the array is huge and arrives as a stream, can you emit peaks online using only O(1) memory (keeping just the last two values)?

