---
type: "practice"
difficulty: "Other"
---

# Fewest End Removals to Discard Both Extremes

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Two Pointers](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Two_Pointers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed array values containing distinct integers. Exactly one element holds the smallest value in the array and exactly one element holds the largest value.

In a single operation you may remove either the leftmost element (the front) or the rightmost element (the back) of the array as it currently stands. Each such removal counts as one operation.

Your job is to erase both the smallest element and the largest element from the array. Return the minimum number of removal operations required so that both of those two elements have been taken out. Any other elements you remove along the way are allowed; only the total count of operations matters.

Because the array holds distinct integers, the positions of the smallest and largest elements are unique and well defined. You only need to reason about how far each of them sits from the two ends.

Input
values: an array of distinct integers.
Output
A single integer: the minimum number of front/back removals needed to eliminate both the minimum and the maximum element.

#### Cons
traints
1 <= values.length <= 10^5
-10^5 <= values[i] <= 10^5
All integers in values are distinct.

#### Example
s & test cases
TEXT
Input: values = [4, 9, 2, 15, 6, 3, 11] -> Output: 4
Why: smallest 2 is at index 2, largest 15 at index 3; removing the first 4 elements from the front clears both — cheaper than working from the back.

Input: values = [1, 5, 8, 3, 9] -> Output: 2
Why: smallest 1 sits at the very front (index 0) and largest 9 at the very back (index 4); pop one from each end for 2 total.

Input: values = [7] -> Output: 1
Why: the lone element is simultaneously the minimum and the maximum, so a single removal handles both.

Input: values = [-3, -8, 0, 5, -1] -> Output: 4
Why: smallest -8 is at index 1 and largest 5 at index 3; every strategy (front-4, back-4, or split 2+2) costs 4 here.

Input: values = [20, 3, 14, 7, 25, 9] -> Output: 4
Why: smallest 3 at index 1, largest 25 at index 4; the split strategy (2 from the front to clear index 1, 2 from the back to clear index 4) wins at 4.

#### Approach

Only the two extreme elements matter, so first find their indices in one pass. Let lo be the smaller index and hi the larger. Since removals only come off the two ends, there are exactly three sensible strategies: (1) remove everything up to and including hi from the front — cost hi + 1; (2) remove everything from lo to the end off the back — cost n - lo; (3) peel the front-side extreme off the front and the back-side extreme off the back — cost (lo + 1) + (n - hi). The optimum is the minimum of these three. This is a greedy/constructive counting argument: any valid removal sequence reduces to one of these three shapes, so comparing them is sufficient.

Complexity

Time: O(n) — a single linear scan locates the positions of the minimum and maximum; the final answer is three arithmetic comparisons. Space: O(1) — only a handful of index variables are kept regardless of input size.

Python
PYTHON
from typing import List

class Solution:
    def minimumDeletions(self, values: List[int]) -> int:
        n = len(values)
        if n == 1:
            return 1

        i_min = min(range(n), key=lambda i: values[i])
        i_max = max(range(n), key=lambda i: values[i])

        lo, hi = min(i_min, i_max), max(i_min, i_max)

        front_both = hi + 1              # remove both from the front
        back_both = n - lo              # remove both from the back
        split = (lo + 1) + (n - hi)     # one extreme from each end

        return min(front_both, back_both, split)
JavaScript
JAVASCRIPT
function minimumDeletions(values) {

#### cons
t n = values.length;
    if (n === 1) return 1;

    let iMin = 0, iMax = 0;
    for (let i = 1; i < n; i++) {
        if (values[i] < values[iMin]) iMin = i;
        if (values[i] > values[iMax]) iMax = i;
    }

#### cons
t lo = Math.min(iMin, iMax);          // nearer the front

#### cons
t hi = Math.max(iMin, iMax);          // nearer the back

#### cons
t frontBoth = hi + 1;                 // both from the front

#### cons
t backBoth = n - lo;                  // both from the back

#### cons
t split = (lo + 1) + (n - hi);        // one extreme from each end

    return Math.min(frontBoth, backBoth, split);
}
Java
JAVA
class Solution {
    public int minimumDeletions(int[] values) {
        int n = values.length;
        if (n == 1) return 1;

        int iMin = 0, iMax = 0;
        for (int i = 1; i < n; i++) {
            if (values[i] < values[iMin]) iMin = i;
            if (values[i] > values[iMax]) iMax = i;
        }

        int lo = Math.min(iMin, iMax);        // nearer the front
        int hi = Math.max(iMin, iMax);        // nearer the back

        int frontBoth = hi + 1;               // both from the front
        int backBoth  = n - lo;               // both from the back
        int split     = (lo + 1) + (n - hi);  // one extreme from each side

        return Math.min(frontBoth, Math.min(backBoth, split));
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int minimumDeletions(vector<int>& values) {
        int n = (int)values.size();
        if (n == 1) return 1;

        int iMin = 0, iMax = 0;
        for (int i = 1; i < n; ++i) {
            if (values[i] < values[iMin]) iMin = i;
            if (values[i] > values[iMax]) iMax = i;
        }

        int lo = min(iMin, iMax);          // closer to the front
        int hi = max(iMin, iMax);          // closer to the back

        int frontBoth = hi + 1;            // clear both from the front
        int backBoth  = n - lo;            // clear both from the back
        int split     = (lo + 1) + (n - hi); // one extreme from each end

        return min({frontBoth, backBoth, split});
    }
};
C
C
#include <stddef.h>

int minimumDeletions(int* values, int valuesSize) {
    int n = valuesSize;
    if (n == 1) return 1;

    int iMin = 0, iMax = 0;
    for (int i = 1; i < n; ++i) {
        if (values[i] < values[iMin]) iMin = i;
        if (values[i] > values[iMax]) iMax = i;
    }

    int lo = iMin < iMax ? iMin : iMax;   /* nearer to the front */
    int hi = iMin < iMax ? iMax : iMin;   /* nearer to the back  */

    int frontBoth = hi + 1;               /* pop both from the front */
    int backBoth  = n - lo;               /* pop both from the back  */
    int split     = (lo + 1) + (n - hi);  /* one from each end       */

    int ans = frontBoth < backBoth ? frontBoth : backBoth;
    if (split < ans) ans = split;
    return ans;
}

#### Follow-up
questions
What changes if the array may contain duplicate values, so the minimum or maximum could appear more than once?
Suppose front removals and back removals have different per-operation costs — minimize total cost instead of count.
Return the exact sequence of front/back operations, not just how many.
Generalize to removing the k smallest and k largest elements with the same end-removal rule.

