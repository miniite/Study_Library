---
type: "practice"
difficulty: "Other"
---

# Locate a Value in a Single-Peak Sequence

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a single-peak sequence elevations: a list of integers that first strictly increases up to one highest element (the peak) and then strictly decreases to the end. Formally, there is an index p with 0 < p < len(elevations) - 1 such that:

elevations[0] < elevations[1] < ... < elevations[p-1] < elevations[p], and
elevations[p] > elevations[p+1] > ... > elevations[len(elevations)-1].

Because the two sides are strictly monotonic, no two adjacent elements are equal, but a value that appears on the way up can appear again on the way down.

Given an integer query, return the minimum index i such that elevations[i] == query. If query does not appear anywhere in the sequence, return -1.

The sequence can be large, so an efficient solution should not scan every element — it should read only about O(log n) positions.

Input

elevations: a single-peak sequence of integers (length >= 3).
query: the integer to locate.

Output

The smallest index whose value equals query, or -1 if there is none.

#### Cons
traints

3 <= len(elevations) <= 10^4
0 <= query <= 10^9
0 <= elevations[i] <= 10^9
elevations is guaranteed to be a valid single-peak sequence (strictly up then strictly down).

#### Example
s & test cases
TEXT
- Input: `elevations = [1,3,5,7,6,4,2], query = 5` -> Output: `2`
  Why: `5` sits on the increasing side at index `2`; that is the first place it occurs.
- Input: `elevations = [1,3,5,7,6,4,2], query = 4` -> Output: `5`
  Why: `4` is absent from the increasing side, so the answer comes from the decreasing side at index `5`.
- Input: `elevations = [2,4,6,4,2], query = 4` -> Output: `1`
  Why: `4` appears on both sides (indices `1` and `3`); the minimum index `1` (the ascending occurrence) must be returned.
- Input: `elevations = [0,2,4,3,1], query = 5` -> Output: `-1`
  Why: `5` is larger than the peak, so it is nowhere in the sequence.
- Input: `elevations = [1,5,2], query = 2` -> Output: `2`
  Why: smallest allowed sequence; `2` is only on the descending side, at the last index.
- Input: `elevations = [10,20,30,25], query = 30` -> Output: `2`
  Why: the query equals the peak value itself, found while binary-searching the ascending half.

#### Approach

Split the problem into three binary searches. First, find the peak: at any index mid, if elevations[mid] < elevations[mid+1] the peak lies to the right, otherwise it lies at mid or to the left — this converges to the single peak in O(log n). The array is now two strictly monotonic runs. Binary-search the ascending run [0, peak] with the usual "go right when too small" rule; because that half is strictly increasing, any match is unique there and (being on the left of the peak) is automatically the smallest index in the whole array, so we check it FIRST and return immediately. If the ascending search fails, binary-search the descending run [peak+1, n-1] with the comparison reversed ("go right when too large"). The pattern is classic binary search on a bitonic/single-peak array, and the left-half-first ordering is what enforces the minimum-index tie-break.

Complexity

Time: O(log n) — three sequential binary searches (peak-finding, ascending half, descending half), each halving its range, so about 3 * log2(n) element reads. Space: O(1) — only a handful of index variables. Justification: strict monotonicity on each side lets standard binary search apply, and the number of accessed positions is logarithmic, satisfying the "don't scan everything" requirement.

Python
PYTHON
from typing import List

class Solution:
    def findInPeakSequence(self, elevations: List[int], query: int) -> int:
        n = len(elevations)
        # 1. locate the peak index
        lo, hi = 0, n - 1
        while lo < hi:
            mid = (lo + hi) // 2
            if elevations[mid] < elevations[mid + 1]:
                lo = mid + 1
            else:
                hi = mid
        peak = lo

        # 2. search the increasing half first (its match is the minimum index)
        lo, hi = 0, peak
        while lo <= hi:
            mid = (lo + hi) // 2
            if elevations[mid] == query:
                return mid
            elif elevations[mid] < query:
                lo = mid + 1
            else:
                hi = mid - 1

        # 3. search the decreasing half (comparison reversed)
        lo, hi = peak + 1, n - 1
        while lo <= hi:
            mid = (lo + hi) // 2
            if elevations[mid] == query:
                return mid
            elif elevations[mid] > query:
                lo = mid + 1
            else:
                hi = mid - 1

        return -1
JavaScript
JAVASCRIPT
function findInPeakSequence(elevations, query) {

#### cons
t n = elevations.length;
    // 1. find the peak index
    let lo = 0, hi = n - 1, mid;
    while (lo < hi) {
        mid = (lo + hi) >> 1;
        if (elevations[mid] < elevations[mid + 1]) lo = mid + 1;
        else hi = mid;
    }

#### cons
t peak = lo;

    // 2. ascending half [0, peak] first -> minimum index
    lo = 0; hi = peak;
    while (lo <= hi) {
        mid = (lo + hi) >> 1;
        if (elevations[mid] === query) return mid;
        else if (elevations[mid] < query) lo = mid + 1;
        else hi = mid - 1;
    }

    // 3. descending half [peak+1, n-1]
    lo = peak + 1; hi = n - 1;
    while (lo <= hi) {
        mid = (lo + hi) >> 1;
        if (elevations[mid] === query) return mid;
        else if (elevations[mid] > query) lo = mid + 1;
        else hi = mid - 1;
    }

    return -1;
}
Java
JAVA
class Solution {
  public int findInPeakSequence(int[] elevations, int query) {
    int n = elevations.length;
    int lo = 0, hi = n - 1, mid, peak;
    // 1. find the peak
    while (lo < hi) {
      mid = lo + (hi - lo) / 2;
      if (elevations[mid] < elevations[mid + 1]) lo = mid + 1;
      else hi = mid;
    }
    peak = lo;
    // 2. ascending half [0, peak] first -> smallest matching index
    lo = 0; hi = peak;
    while (lo <= hi) {
      mid = lo + (hi - lo) / 2;
      if (elevations[mid] == query) return mid;
      else if (elevations[mid] < query) lo = mid + 1;
      else hi = mid - 1;
    }
    // 3. descending half [peak+1, n-1]
    lo = peak + 1; hi = n - 1;
    while (lo <= hi) {
      mid = lo + (hi - lo) / 2;
      if (elevations[mid] == query) return mid;
      else if (elevations[mid] > query) lo = mid + 1;
      else hi = mid - 1;
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
  int findInPeakSequence(const vector<int>& elevations, int query) {
    int n = (int)elevations.size();
    int lo = 0, hi = n - 1, mid, peak;
    // 1. find the peak index
    while (lo < hi) {
      mid = lo + (hi - lo) / 2;
      if (elevations[mid] < elevations[mid + 1]) lo = mid + 1;
      else hi = mid;
    }
    peak = lo;
    // 2. search the strictly increasing left half first (guarantees minimum index)
    lo = 0; hi = peak;
    while (lo <= hi) {
      mid = lo + (hi - lo) / 2;
      if (elevations[mid] == query) return mid;
      else if (elevations[mid] < query) lo = mid + 1;
      else hi = mid - 1;
    }
    // 3. search the strictly decreasing right half
    lo = peak + 1; hi = n - 1;
    while (lo <= hi) {
      mid = lo + (hi - lo) / 2;
      if (elevations[mid] == query) return mid;
      else if (elevations[mid] > query) lo = mid + 1;
      else hi = mid - 1;
    }
    return -1;
  }
};
C
C
int findInPeakSequence(int* elevations, int elevationsSize, int query) {
    int lo = 0, hi = elevationsSize - 1, mid, peak;
    /* 1. locate the peak: keep the higher neighbour's side */
    while (lo < hi) {
        mid = lo + (hi - lo) / 2;
        if (elevations[mid] < elevations[mid + 1]) lo = mid + 1;
        else hi = mid;
    }
    peak = lo;
    /* 2. ascending half [0, peak] is increasing -> normal binary search (checked first for min index) */
    lo = 0; hi = peak;
    while (lo <= hi) {
        mid = lo + (hi - lo) / 2;
        if (elevations[mid] == query) return mid;
        else if (elevations[mid] < query) lo = mid + 1;
        else hi = mid - 1;
    }
    /* 3. descending half [peak+1, n-1] is decreasing -> reversed comparison */
    lo = peak + 1; hi = elevationsSize - 1;
    while (lo <= hi) {
        mid = lo + (hi - lo) / 2;
        if (elevations[mid] == query) return mid;
        else if (elevations[mid] > query) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

#### Follow-up
questions
The real setting restricts you to a limited number of index reads (say 100). Prove your access count stays within that budget for n up to 10^4.
What if the sequence could have flat plateaus (adjacent equal values) instead of being strictly monotonic — how does that break plain binary search, and how would you adapt?
Return the peak index alone, or count how many elements are strictly greater than query.
Generalize to a sequence with several peaks and valleys — can you still do better than a linear scan?

