---
type: "practice"
difficulty: "Other"
---

# Second Largest Distinct Value

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Arrays](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Arrays.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array of integers amounts. Values may repeat and may be negative.

Return the second largest distinct value in the array. Formally, take the set of all values that appear (removing duplicates), sort that set in strictly decreasing order, and return the value in the second position.

If there is no such value — the array is empty, has a single element, or every element is equal so that fewer than two distinct values exist — there is no second largest value. In that case return null (None in Python). The reference C/C++/Java solutions signal this "not found" case explicitly (via an optional/nullable return or a boolean out-parameter).

Aim for a single pass over the array using only O(1) extra space.

#### Cons
traints

0 <= amounts.length <= 100000
-1000000000 <= amounts[i] <= 1000000000
"Distinct" means by value: repeated equal values count as one when determining the ranking.

#### Example
s & test cases
TEXT
Input: [40, 10, 30, 30, 20] -> Output: 30
Why: distinct values sorted descending are 40, 30, 20, 10; the second one is 30 (duplicates of 30 collapse to one).

Input: [7] -> Output: null
Why: only one value exists, so there is no second largest.

Input: [5, 5, 5] -> Output: null
Why: three elements but only one distinct value, so no second largest.

Input: [] -> Output: null
Why: an empty array has no first, let alone second, largest value.

Input: [-2, -5, -2, -9] -> Output: -5
Why: distinct values descending are -2, -5, -9; the second is -5, confirming negatives are handled.

Input: [1000000000, -1000000000] -> Output: -1000000000
Why: exactly two distinct values at the extreme bounds; the smaller of the two is the second largest.

#### Approach

Track the two largest distinct values in one linear scan. Keep hi (largest seen) and lo (second largest distinct seen). For each element x: if x beats hi, the old hi slides down into lo and x becomes the new hi; otherwise, if x is different from hi and larger than the current lo (or lo is still unset), x becomes the new lo. The x != hi guard is what enforces distinctness, so repeated copies of the maximum never populate the runner-up. If lo never gets set, fewer than two distinct values existed and we report "no answer". In relational terms this is SELECT MAX(v) FROM t WHERE v < (SELECT MAX(v) FROM t), but the two-variable scan computes it in a single O(n) pass instead of two aggregate passes. The same idea generalizes to the k-th largest distinct value by keeping k running slots (or a size-k min-heap).

Complexity

Time: O(n) — a single pass over the array. Space: O(1) — only two running values (the largest and the second largest distinct value seen so far) are stored regardless of input size.

Python
PYTHON
def second_largest_distinct(amounts):
    """Return the second largest distinct value, or None if it does not exist."""
    hi = lo = None
    for x in amounts:
        if hi is None or x > hi:        # new maximum; old max becomes runner-up
            lo, hi = hi, x
        elif x != hi and (lo is None or x > lo):
            lo = x                       # new distinct value strictly below the max
    return lo
JavaScript
JAVASCRIPT
function secondLargestDistinct(amounts) {
  // Returns null when fewer than two distinct values exist.
  let hi = null, lo = null;
  for (const x of amounts) {
    if (hi === null || x > hi) {          // new maximum; old max becomes runner-up
      lo = hi;
      hi = x;
    } else if (x !== hi && (lo === null || x > lo)) {
      lo = x;                              // new distinct value strictly below the max
    }
  }
  return lo;
}
Java
JAVA
class Solution {
    // Returns null when fewer than two distinct values exist.
    public Long secondLargestDistinct(long[] amounts) {
        Long hi = null, lo = null;
        for (long x : amounts) {
            if (hi == null || x > hi) {          // new maximum; old max is runner-up
                lo = hi;
                hi = x;
            } else if (x != hi && (lo == null || x > lo)) {
                lo = x;                           // new distinct value below the max
            }
        }
        return lo;
    }
}
C++
CPP
#include <vector>
#include <optional>
using namespace std;

// Returns nullopt when fewer than two distinct values exist.
optional<long long> secondLargestDistinct(const vector<long long>& amounts) {
    optional<long long> hi, lo;
    for (long long x : amounts) {
        if (!hi || x > *hi) {           // new maximum; old max becomes runner-up
            lo = hi;
            hi = x;
        } else if (x != *hi && (!lo || x > *lo)) {
            lo = x;                      // new distinct value strictly below the max
        }
    }
    return lo;
}
C
C
#include <stdbool.h>

/* Returns the second largest distinct value.
   *found is set to false when fewer than two distinct values exist. */
long long secondLargestDistinct(const long long *amounts, int n, bool *found) {
    bool hasHi = false, hasLo = false;
    long long hi = 0, lo = 0;
    for (int i = 0; i < n; i++) {
        long long x = amounts[i];
        if (!hasHi || x > hi) {
            lo = hi; hasLo = hasHi;   /* old max becomes the runner-up */
            hi = x; hasHi = true;
        } else if (x != hi && (!hasLo || x > lo)) {
            lo = x; hasLo = true;     /* new distinct value below the max */
        }
    }
    *found = hasLo;
    return lo;
}

#### Follow-up
questions
Generalize to the k-th largest distinct value — how does the approach and complexity change (k running slots or a size-k heap)?
Return the second largest value counting duplicates instead of distinct values — what changes?
The data arrives as a stream and you must answer at any moment — maintain the two values online.
The array is enormous and distributed across machines — how do you compute the second largest distinct value with minimal communication?

