---
type: "practice"
difficulty: "Other"
---

# Peak Influence Rating from Sorted Endorsements

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A writer has published a collection of articles. For each article you are given the number of endorsements it received as an integer array endorsements, where endorsements[i] is the endorsement count of the i-th article. The array is sorted in non-descending (ascending) order.

Define the writer's influence rating as the largest integer h such that at least h of the articles have each received at least h endorsements. Return this influence rating.

Your algorithm must run in logarithmic time with respect to the number of articles.

Input: An ascending-sorted integer array endorsements.

Output: A single integer — the influence rating h.

#### Cons
traints:

n == endorsements.length
1 <= n <= 10^5
0 <= endorsements[i] <= 1000
endorsements is sorted in ascending order.

#### Example
s & test cases
TEXT
Input: endorsements = [0,2,3,4,7] -> Output: 3
Why: 3 articles (with 3, 4, 7 endorsements) each have at least 3 endorsements, but there are not 4 articles with at least 4 each.

Input: endorsements = [2,4,50] -> Output: 2
Why: All 3 articles have >= 2, so h >= 2; but only 2 articles have >= 3, and we cannot reach h = 3 because we'd need 3 articles with >= 3.

Input: endorsements = [0] -> Output: 0
Why: The single article has 0 endorsements, so no h >= 1 is achievable.

Input: endorsements = [8] -> Output: 1
Why: Only one article exists; at least 1 article (this one) has >= 1 endorsement, so h = 1 (cannot be 2 with a single article).

Input: endorsements = [3,3,3,3] -> Output: 3
Why: All 4 articles have >= 3, giving h = 3; h = 4 would require 4 articles each with >= 4, which fails since each has only 3.

Input: endorsements = [0,0,0] -> Output: 0
Why: No article has any endorsement, so the rating is 0 (edge case: all zeros).

#### Approach

Because the array is sorted ascending, choosing an index m (0-based) means the n - m articles from m to the end each have at least endorsements[m] endorsements. So the predicate endorsements[m] >= n - m certifies a candidate rating of n - m. As m grows, n - m shrinks and the predicate flips from false to true exactly once, making it monotonic. Binary search for the smallest index m where the predicate holds; the answer is n - m. This is the classic "binary search on a monotonic predicate over a sorted array" pattern.

Complexity

Time: O(log n) — binary search halves the candidate index range each iteration. Space: O(1) — only a few index variables are used.

Python
PYTHON
from typing import List

class Solution:
    def influenceRating(self, endorsements: List[int]) -> int:
        n = len(endorsements)
        lo, hi = 0, n
        while lo < hi:
            mid = (lo + hi) // 2
            # n - mid articles from index mid onward each have >= endorsements[mid]
            if endorsements[mid] >= n - mid:
                hi = mid
            else:
                lo = mid + 1
        return n - lo
JavaScript
JAVASCRIPT
function influenceRating(endorsements) {

#### cons
t n = endorsements.length;
    let lo = 0, hi = n;
    while (lo < hi) {

#### cons
t mid = lo + Math.floor((hi - lo) / 2);
        if (endorsements[mid] >= n - mid) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    return n - lo;
}
Java
JAVA
class Solution {
    public int influenceRating(int[] endorsements) {
        int n = endorsements.length;
        int lo = 0, hi = n;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (endorsements[mid] >= n - mid)
                hi = mid;
            else
                lo = mid + 1;
        }
        return n - lo;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
 public:
  int influenceRating(vector<int>& endorsements) {
    int n = endorsements.size();
    int lo = 0, hi = n;
    while (lo < hi) {
      int mid = lo + (hi - lo) / 2;
      if (endorsements[mid] >= n - mid)
        hi = mid;          // this index and everything to its right qualify
      else
        lo = mid + 1;      // too few large values, look right
    }
    return n - lo;
  }
};
C
C
#include <stddef.h>

int influenceRating(int* endorsements, int n) {
    int lo = 0, hi = n;            /* search for smallest valid index m */
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        /* n - mid articles from index mid onward each have >= endorsements[mid] */
        if (endorsements[mid] >= n - mid)
            hi = mid;
        else
            lo = mid + 1;
    }
    return n - lo;
}

#### Follow-up
questions
What if the array is NOT sorted? Discuss sorting first (O(n log n)) or a counting-bucket O(n) approach.
Can you achieve O(n) without binary search using a frequency array capped at n?
If endorsement counts stream in one at a time and cannot fit in memory, how would you maintain the rating?
Return the actual set of articles that form the "h-core" (the h articles each with >= h endorsements).

