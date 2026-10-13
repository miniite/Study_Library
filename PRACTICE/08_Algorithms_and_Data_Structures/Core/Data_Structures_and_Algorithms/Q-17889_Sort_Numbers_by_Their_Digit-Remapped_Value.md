---
type: "practice"
difficulty: "Other"
---

# Sort Numbers by Their Digit-Remapped Value

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed integer array dial of length 10 that describes a remapping of the decimal digits: dial[d] = e means the digit d should be replaced by the digit e. All ten entries of dial are distinct, so dial is a permutation of the digits 0 through 9.

The remapped value of a non-negative integer is obtained by replacing every digit d in its ordinary base-10 representation with dial[d], and then reading the result as an ordinary integer. Any leading zeros introduced by the remapping are dropped. For example, if dial maps 3 -> 0 and 8 -> 7, then the number 38 becomes "07", which is the integer 7.

You are also given an integer array values. Return the array values sorted in non-decreasing order according to the remapped value of each element.

Notes:

Two elements whose remapped values are equal must keep the same relative order they had in the input (the ordering is stable).
The elements returned are the original numbers; they are only ordered by, not replaced by, their remapped values.

#### Cons
traints:

dial.length == 10
0 <= dial[d] <= 9
All entries of dial are distinct.
1 <= values.length <= 3 * 10^4
0 <= values[i] < 10^9

#### Example
s & test cases
TEXT
Input: dial = [1,0,2,3,4,5,6,7,8,9], values = [10,1,100] -> Output: [1,10,100]
Why: this dial only swaps 0 and 1. 10 -> "01" = 1, 1 -> "0" = 0, 100 -> "011" = 11. Ordering by (0,1,11) gives [1,10,100].

Input: dial = [0,1,2,3,4,5,6,7,8,9], values = [70,8,25] -> Output: [8,25,70]
Why: the identity dial leaves every number unchanged, so this is a plain numeric sort.

Input: dial = [4,5,0,1,2,3,6,7,8,9], values = [233,500,33] -> Output: [233,33,500]
Why: 233 -> "011" = 11 and 33 -> "11" = 11 tie; stability keeps 233 before 33. 500 -> "344" = 344 comes last.

Input: dial = [9,8,7,6,5,4,3,2,1,0], values = [12,21] -> Output: [21,12]
Why: reverse dial. 12 -> "87" = 87, 21 -> "78" = 78, so 21 precedes 12.

Input: dial = [2,3,4,5,6,7,8,9,0,1], values = [0] -> Output: [0]
Why: single-element edge case; 0 remaps to dial[0] = 2, and a one-element array is already sorted.

#### Approach

This is a "sort by a derived key" (Schwartzian transform / decorate-sort-undecorate) problem. For each number, precompute its remapped value once by walking its digits from most significant to least significant, substituting dial[d] and rebuilding the integer (which naturally discards leading zeros). Then sort the original numbers using that precomputed value as the sort key. Precomputing avoids recomputing the mapping inside every comparison. Stability is required, so either use a stable sort (Python/Java object sort) or explicitly break ties on the original index (C/C++/JS) so equal-keyed elements retain input order. Remapped values can reach 9 digits, which fits in a 32-bit int, but a 64-bit type is used for safety.

Complexity

Time: O(n log n), where n = values.length. Computing a remapped value scans at most 10 digits (O(1) per element, O(n) total), and the dominating cost is the comparison sort. Space: O(n) for the auxiliary keys / index array.

Python
PYTHON
class Solution:
    def sortJumbled(self, dial, values):
        def remap(x):
            if x == 0:
                return dial[0]
            digs = []
            while x > 0:
                digs.append(x % 10)
                x //= 10
            v = 0
            for d in reversed(digs):
                v = v * 10 + dial[d]
            return v
        # Python's sorted is stable, so keying on the remapped value
        # preserves the input order among equal keys.
        return sorted(values, key=remap)
JavaScript
JAVASCRIPT
function sortJumbled(dial, values) {

#### cons
t remap = (x) => {
        if (x === 0) return dial[0];

#### cons
t digs = [];
        while (x > 0) { digs.push(x % 10); x = Math.floor(x / 10); }
        let v = 0;
        for (let i = digs.length - 1; i >= 0; i--) v = v * 10 + dial[digs[i]];
        return v;
    };

#### cons
t mapped = values.map(remap);

#### cons
t order = values.map((_, i) => i);
    order.sort((a, b) => (mapped[a] - mapped[b]) || (a - b)); // tie-break => stable
    return order.map(i => values[i]);
}
Java
JAVA
import java.util.*;

class Solution {
    public int[] sortJumbled(int[] dial, int[] values) {
        int n = values.length;
        long[] mapped = new long[n];
        for (int i = 0; i < n; i++) mapped[i] = remap(values[i], dial);

        Integer[] order = new Integer[n];
        for (int i = 0; i < n; i++) order[i] = i;
        Arrays.sort(order, (a, b) -> {
            if (mapped[a] != mapped[b]) return Long.compare(mapped[a], mapped[b]);
            return Integer.compare(a, b);   // tie-break => stable
        });

        int[] res = new int[n];
        for (int i = 0; i < n; i++) res[i] = values[order[i]];
        return res;
    }

    private long remap(int x, int[] dial) {
        if (x == 0) return dial[0];
        int[] digs = new int[10];
        int m = 0;
        while (x > 0) { digs[m++] = x % 10; x /= 10; }
        long v = 0;
        for (int i = m - 1; i >= 0; i--) v = v * 10 + dial[digs[i]];
        return v;
    }
}
C++
CPP
#include <vector>
#include <array>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<int> sortJumbled(vector<int>& dial, vector<int>& values) {
        int n = values.size();
        auto remap = [&] (int x) -> long long {
            if (x == 0) return dial[0];
            int digs[10], m = 0;
            while (x > 0) { digs[m++] = x % 10; x /= 10; }
            long long v = 0;
            for (int i = m - 1; i >= 0; i--) v = v * 10 + dial[digs[i]];
            return v;
        };
        vector<array<long long, 2>> keyed(n); // {mappedValue, originalIndex}
        for (int i = 0; i < n; i++) keyed[i] = { remap(values[i]), (long long)i };
        sort(keyed.begin(), keyed.end());     // lexicographic => stable by index
        vector<int> res(n);
        for (int i = 0; i < n; i++) res[i] = values[keyed[i][1]];
        return res;
    }
};
C
C
#include <stdlib.h>

typedef struct { long long mapped; int idx; int val; } Item;

static long long remapValue(int x, int* dial) {
    if (x == 0) return dial[0];
    int digs[10], n = 0;
    while (x > 0) { digs[n++] = x % 10; x /= 10; }
    long long v = 0;
    for (int i = n - 1; i >= 0; i--) v = v * 10 + dial[digs[i]];
    return v;
}

static int cmpItem(const void* a, const void* b) {

#### cons
t Item* x = (const Item*)a;

#### cons
t Item* y = (const Item*)b;
    if (x->mapped != y->mapped) return (x->mapped < y->mapped) ? -1 : 1;
    return x->idx - y->idx;   /* tie-break on original index => stable */
}

int* sortJumbled(int* dial, int dialSize, int* values, int valuesSize, int* returnSize) {
    Item* items = malloc(sizeof(Item) * valuesSize);
    for (int i = 0; i < valuesSize; i++) {
        items[i].mapped = remapValue(values[i], dial);
        items[i].idx = i;
        items[i].val = values[i];
    }
    qsort(items, valuesSize, sizeof(Item), cmpItem);
    int* res = malloc(sizeof(int) * valuesSize);
    for (int i = 0; i < valuesSize; i++) res[i] = items[i].val;
    free(items);
    *returnSize = valuesSize;
    return res;
}

#### Follow-up
questions
Why is stability essential here, and what breaks if you use an unstable sort without the index tie-break?
Instead of remapping to a numeric value, remap into a fixed-width digit string and sort lexicographically — when do the two approaches diverge?
The numbers fit in 32/64-bit integers; how would you handle arbitrarily long numbers given as strings so the remapped value can't overflow?
Return the k smallest by remapped value without fully sorting — can you do better than O(n log n)?

