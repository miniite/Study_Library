---
type: "practice"
difficulty: "Other"
---

# Total Insertion Cost of a Sorted Multiset

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Segment Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Segment_Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an integer array readings. Process its elements strictly left to right and insert each one, in order, into a collection that is always kept sorted (duplicates are allowed, so it is a multiset).

Inserting a value r has a cost equal to the minimum of these two quantities, both measured over the values already inserted before this step:

the count of already-inserted values that are strictly less than r, and
the count of already-inserted values that are strictly greater than r.

For instance, if the collection currently holds [2, 4, 6, 10] and you insert 6, then two values (2, 4) are strictly less and one value (10) is strictly greater, so the cost of that insertion is min(2, 1) = 1, after which the collection becomes [2, 4, 6, 6, 10].

Return the total insertion cost summed over every element of readings. Because the total can be large, return it modulo 1_000_000_007.

Input

readings: an integer array processed in the given order.

Output

A single integer: the total insertion cost taken modulo 1_000_000_007.

#### Cons
traints

1 <= readings.length <= 10^5
1 <= readings[i] <= 10^5

Note: equal values already present count as neither strictly less nor strictly greater, so they never contribute to either side of the min.

#### Example
s & test cases
TEXT
Input: readings = [1,5,6,2] -> Output: 1
Why: Costs are 0 (empty), 0 (min(1,0)), 0 (min(2,0)), then inserting 2 gives min(1 smaller, 2 larger) = 1. Total = 1.

Input: readings = [1,2,3,6,5,4] -> Output: 3
Why: The first four cost 0; inserting 5 costs min(3,1)=1; inserting 4 costs min(3,2)=2. Total = 0+0+0+0+1+2 = 3.

Input: readings = [1,3,3,3,2,4,2,1,2] -> Output: 4
Why: Non-zero costs occur at the 2 after three 3's (min(1,3)=1), the second 2 (min(1,4)=1), and the final 2 (min(2,4)=2); everything else is 0. Total = 4. Exercises heavy duplicates.

Input: readings = [5] -> Output: 0
Why: A single element has nothing inserted before it, so both counts are 0. Minimal-length edge case.

Input: readings = [2,2,2,2] -> Output: 0
Why: Every insert sees only equal values, which are neither strictly smaller nor strictly larger, so each cost is min(0,0)=0. All-duplicates edge case.

#### Approach

For every incoming value you need two order statistics restricted to the elements seen so far: how many are strictly smaller and how many are strictly larger. A Binary Indexed Tree (Fenwick tree) indexed by value supports both in O(log M): query(x-1) gives the count strictly less than x, and i - query(x) gives the count strictly greater (where i is how many elements have already been inserted, and query(x) counts those <= x). Add min(less, greater) to the running total, then point-update the tree at x. The underlying pattern is "process online + count inversions / rank queries with a Fenwick tree" (a merge-sort BIT or an order-statistics balanced tree would work equally well).

Complexity

Time: O(n log M), where n is the number of readings and M is the maximum value; each element performs two prefix queries and one point update, each O(log M) on the Fenwick tree. Space: O(M) for the tree. (Coordinate compression can replace M with the number of distinct values when the range is sparse.)

Python
PYTHON
def total_insertion_cost(readings):
    MOD = 10 ** 9 + 7
    n = len(readings)
    max_val = max(readings)
    tree = [0] * (max_val + 1)          # Fenwick tree over 1..max_val

    def query(i):                        # prefix count of values <= i
        s = 0
        while i > 0:
            s += tree[i]
            i -= i & (-i)
        return s

    def update(i):                       # insert one occurrence of value i
        while i <= max_val:
            tree[i] += 1
            i += i & (-i)

    total = 0
    for i, x in enumerate(readings):
        less = query(x - 1)              # strictly < x
        greater = i - query(x)           # strictly > x  (i already inserted so far)
        total = (total + min(less, greater)) % MOD
        update(x)
    return total
JavaScript
JAVASCRIPT
/**
 * @param {number[]} readings
 * @return {number}
 */
function totalInsertionCost(readings) {

#### cons
t MOD = 1000000007n;

#### cons
t n = readings.length;

#### cons
t maxVal = Math.max(...readings);

#### cons
t tree = new Array(maxVal + 1).fill(0); // Fenwick tree over 1..maxVal

    let total = 0n;
    for (let i = 0; i < n; i++) {

#### cons
t x = readings[i];
        let less = 0, leq = 0;
        for (let j = x - 1; j > 0; j -= j & (-j)) less += tree[j]; // strictly < x
        for (let j = x;     j > 0; j -= j & (-j)) leq  += tree[j]; // <= x

#### cons
t greater = i - leq;                                   // strictly > x
        total = (total + BigInt(Math.min(less, greater))) % MOD;
        for (let j = x; j <= maxVal; j += j & (-j)) tree[j] += 1;   // insert x
    }
    return Number(total);
}
Java
JAVA
class Solution {
    public int totalInsertionCost(int[] readings) {
        final long MOD = 1000000007L;
        int n = readings.length;
        int maxVal = 0;
        for (int v : readings) maxVal = Math.max(maxVal, v);

        long[] tree = new long[maxVal + 1];   // Fenwick tree over 1..maxVal
        long total = 0;
        for (int i = 0; i < n; i++) {
            int x = readings[i];
            long less = 0, leq = 0;
            for (int j = x - 1; j > 0; j -= j & (-j)) less += tree[j]; // strictly < x
            for (int j = x;     j > 0; j -= j & (-j)) leq  += tree[j]; // <= x
            long greater = (long) i - leq;                            // strictly > x
            total = (total + Math.min(less, greater)) % MOD;
            for (int j = x; j <= maxVal; j += j & (-j)) tree[j] += 1;  // insert x
        }
        return (int) total;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int totalInsertionCost(vector<int>& readings) {

#### cons
t long long MOD = 1000000007LL;
        int n = (int)readings.size();
        int maxVal = *max_element(readings.begin(), readings.end());
        vector<long long> tree(maxVal + 1, 0);   // Fenwick tree over 1..maxVal

        long long total = 0;
        for (int i = 0; i < n; i++) {
            int x = readings[i];
            long long less = 0, leq = 0;
            for (int j = x - 1; j > 0; j -= j & (-j)) less += tree[j]; // strictly < x
            for (int j = x;     j > 0; j -= j & (-j)) leq  += tree[j]; // <= x
            long long greater = (long long)i - leq;                   // strictly > x
            total = (total + min(less, greater)) % MOD;
            for (int j = x; j <= maxVal; j += j & (-j)) tree[j] += 1;  // insert x
        }
        return (int)total;
    }
};
C
C
#include <stdlib.h>

#define MODc 1000000007LL

/* Fenwick (Binary Indexed Tree) over the value domain 1..maxVal. */
long long totalInsertionCost(int *readings, int n) {
    int maxVal = 0;
    for (int i = 0; i < n; i++)
        if (readings[i] > maxVal) maxVal = readings[i];

    long long *tree = (long long *)calloc(maxVal + 1, sizeof(long long));
    long long total = 0;
    for (int i = 0; i < n; i++) {
        int x = readings[i];
        long long less = 0, leq = 0;          /* counts among the i already inserted */
        for (int j = x - 1; j > 0; j -= j & (-j)) less += tree[j];  /* strictly < x */
        for (int j = x;     j > 0; j -= j & (-j)) leq  += tree[j];  /* <= x        */
        long long greater = (long long)i - leq;                    /* strictly > x */
        total = (total + (less < greater ? less : greater)) % MODc;
        for (int j = x; j <= maxVal; j += j & (-j)) tree[j] += 1;   /* insert x     */
    }
    free(tree);
    return total;
}

#### Follow-up
questions
The values can be up to 10^9 but the array has at most 10^5 elements. How do you keep the Fenwick tree small? (Coordinate-compress the distinct values first.)
Solve it without a Fenwick tree, using a modified merge sort that counts, for each element, how many later-processed items are smaller/larger. What is the trade-off?
Suppose the cost were max of the two counts instead of min — does the same structure still work, and what changes?
If elements could also be removed from the multiset over time (a mix of inserts and deletes), how would you maintain the same rank queries efficiently?

