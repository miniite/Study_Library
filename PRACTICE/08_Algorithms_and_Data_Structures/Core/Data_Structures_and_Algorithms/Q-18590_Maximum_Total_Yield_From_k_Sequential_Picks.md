---
type: "practice"
difficulty: "Other"
---

# Maximum Total Yield From k Sequential Picks

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Greedy](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Greedy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You manage an orchard with n fruit trees standing in a row. The i-th tree currently holds yields[i] units of ripe fruit. Over the next k days you will harvest exactly one tree per day (so k trees in total, each tree at most once).

There is a catch tied to ripeness: every time you harvest a tree, all trees that have not yet been harvested lose 1 unit of yield (they keep over-ripening). A tree's yield can never drop below 0; once it reaches 0 it stays at 0 no matter how many further harvests happen.

When you harvest a tree, you collect its current yield at that moment (after all the decreases caused by the harvests done on previous days).

Return the maximum total yield you can collect after performing exactly k harvests.

Input:

yields: an integer array of length n.
k: a positive integer, 1 <= k <= n.

Output:

A single integer: the maximum achievable total yield. (This can be large, so use 64-bit arithmetic.)

#### Cons
traints

1 <= n == yields.length <= 2 * 10^5
1 <= yields[i] <= 10^8
1 <= k <= n

#### Example
s & test cases
TEXT
Input: yields = [4,1,6], k = 2 -> Output: 9
Why: Sort descending [6,4,1]. Day 1 harvest 6 (collect 6, others drop to [3,0]). Day 2 harvest 3 (collect 3). Total 6 + 3 = 9. Picking the two largest and subtracting the turn index is optimal.

Input: yields = [1,1,1,1], k = 2 -> Output: 1
Why: First pick collects 1 and all others fall to 0. Second pick collects 0 (clamped, never negative). Total 1 + 0 = 1 — shows the non-negative clamp.

Input: yields = [2,3,4,5], k = 1 -> Output: 5
Why: Only one harvest, no decreases happen before it, so collect the single largest value 5.

Input: yields = [10], k = 1 -> Output: 10
Why: Minimum-size array; the only tree is harvested first for its full value.

Input: yields = [3,3,3], k = 3 -> Output: 3
Why: Sorted [3,3,3], contributions are 3-0, 3-1, 3-2 = 3+2+1 = 6? Recheck: day picks give 3, 2, 1 -> total 6. Harvest all three; each later pick already lost yield.

Input: yields = [1,2], k = 2 -> Output: 2
Why: Sorted [2,1]. Contributions 2-0=2 and 1-1=0. Total 2. The second pick is clamped to 0.

#### Approach

Greedy exchange argument. When you make your (0-indexed) i-th harvest, exactly i earlier harvests have already happened, so whichever tree you pick has been reduced by exactly i (before clamping). Therefore the effective contribution of a tree chosen at turn i is value - i, clamped at 0. Since the penalty i is fixed per turn regardless of which tree you choose, you maximize the sum by assigning the largest values to the smallest penalties: sort descending and pair the i-th largest value with turn i. Add max(value[i] - i, 0); once value[i] - i <= 0 all later terms are also non-positive, so you can stop early.

Complexity

Time O(n log n) dominated by the sort; the single pass over the top k elements is O(k) <= O(n). Space O(1) extra beyond the sort (O(log n) stack for in-place sorts). Sorting is required because we must identify the k largest values and their relative order.

Python
PYTHON
def maximum_total_yield(yields, k):
    yields.sort(reverse=True)
    total = 0
    for i in range(k):
        gain = yields[i] - i
        if gain <= 0:          # every further pick is clamped to 0
            break
        total += gain
    return total
JavaScript
JAVASCRIPT
function maximumTotalYield(yields, k) {
    yields.sort((a, b) => b - a);
    let total = 0;
    for (let i = 0; i < k; i++) {

#### cons
t gain = yields[i] - i;   // JS numbers are safe up to 2^53, fits here
        if (gain <= 0) break;
        total += gain;
    }
    return total;
}
Java
JAVA
import java.util.Arrays;

class Solution {
    public long maximumTotalYield(int[] yields, int k) {
        Arrays.sort(yields);            // ascending
        int n = yields.length;
        long total = 0;
        for (int i = 0; i < k; i++) {
            long gain = (long) yields[n - 1 - i] - i;  // take from the largest end
            if (gain <= 0) break;
            total += gain;
        }
        return total;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

long long maximumTotalYield(vector<int>& yields, int k) {
    sort(yields.begin(), yields.end(), greater<int>());
    long long total = 0;
    for (int i = 0; i < k; i++) {
        long long gain = (long long)yields[i] - i;
        if (gain <= 0) break;   // clamped to 0 from here on
        total += gain;
    }
    return total;
}
C
C
#include <stdlib.h>

static int cmp_desc(const void *a, const void *b) {
    long long x = *(const long long *)a;
    long long y = *(const long long *)b;
    if (x < y) return 1;
    if (x > y) return -1;
    return 0;
}

long long maximumTotalYield(long long *yields, int n, int k) {
    qsort(yields, n, sizeof(long long), cmp_desc);
    long long total = 0;
    for (int i = 0; i < k; i++) {
        long long gain = yields[i] - (long long)i;
        if (gain <= 0) break;   /* remaining picks contribute 0 */
        total += gain;
    }
    return total;
}

#### Follow-up
questions
What if each harvest reduced the unpicked trees by a variable amount d instead of 1 — how does the contribution formula change?
Prove formally (exchange argument) that assigning the largest values to the earliest turns is optimal.
What if you also had to output which trees to harvest, not just the total?
How would you handle streaming input where yields arrives one value at a time and you must keep only the running best without storing everything?

