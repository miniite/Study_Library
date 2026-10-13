---
type: "practice"
difficulty: "Other"
---

# Partition Values into Consecutive Runs

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Greedy](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Greedy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an integer array values and a positive integer runLength. You want to reorganize all of the array's elements into one or more groups so that:

every group contains exactly runLength elements, and
the elements of each group can be arranged as runLength consecutive integers, i.e. a sequence of the form x, x+1, x+2, ..., x+runLength-1.

Every element must be placed in exactly one group. Duplicate values are allowed, and each occurrence is treated as a separate item that must be placed independently.

Return true if the whole array can be split into such groups, and false otherwise.

Input: an integer array values, and an integer runLength.

Output: a boolean — true if a valid partition into consecutive runs exists, else false.

#### Cons
traints:

1 <= values.length <= 10^4
0 <= values[i] <= 10^9
1 <= runLength <= values.length

Note: a valid partition uses every element, so if values.length is not divisible by runLength, the answer is immediately false. When runLength == 1, every element forms its own (trivially consecutive) group, so the answer is always true.

#### Example
s & test cases
TEXT
Input: values = [4,5,6,9,5,6,7,10,11], runLength = 3 -> Output: true
Why: the nine items split into [4,5,6], [5,6,7], and [9,10,11], three runs of three consecutive numbers.

Input: values = [3,4,5,6,7], runLength = 4 -> Output: false
Why: 5 elements cannot be evenly divided into groups of 4, so no valid partition exists.

Input: values = [1,1,2,2,3,3], runLength = 3 -> Output: true
Why: duplicates are fine — the six items form two identical runs [1,2,3] and [1,2,3].

Input: values = [1,2,3,3], runLength = 2 -> Output: false
Why: pairing 1 with 2 leaves the two 3's, which are not consecutive, so no assignment works.

Input: values = [8], runLength = 1 -> Output: true
Why: with runLength 1 a single element is its own consecutive run.

Input: values = [10,11,13,14], runLength = 2 -> Output: true
Why: the values split into [10,11] and [13,14]; a gap between the pairs is allowed since only within-group consecutiveness matters.

#### Approach

This is a greedy + counting problem. Count how many times each value occurs and consider the distinct values in ascending order. The smallest value that still has a positive remaining count cannot be the middle or tail of any run (nothing smaller is left), so it must be the start of one or more runs — exactly base = count[smallest] of them. Each such run also needs one copy of smallest+1, smallest+2, ..., smallest+runLength-1, so decrement each of those runLength-1 values by base; if any of them has fewer than base copies available, a valid partition is impossible. Repeat, always anchoring on the smallest live value. Using an ordered map (or a hash count over a sorted key list) makes "smallest live value" cheap, and each item is consumed exactly once.

Complexity

Time: O(n log n), where n is the number of elements. Building the ordered counts (sorting or an ordered map) costs O(n log n), and that dominates. The greedy sweep is only O(n) overall: a value that opens base runs does runLength decrement steps while consuming base * runLength items, so total decrement work equals the item count. Space: O(n) for the counts of the distinct values.

Python
PYTHON
from collections import Counter
from typing import List

class Solution:
    def canPartitionRuns(self, values: List[int], runLength: int) -> bool:
        if len(values) % runLength != 0:
            return False

        count = Counter(values)
        for start in sorted(count):          # smallest remaining value first
            base = count[start]
            if base > 0:                     # this many runs must begin at `start`
                for k in range(runLength):
                    v = start + k
                    if count[v] < base:      # missing consecutive items
                        return False
                    count[v] -= base
        return True
JavaScript
JAVASCRIPT
function canPartitionRuns(values, runLength) {
    if (values.length % runLength !== 0) return false;

#### cons
t count = new Map();
    for (const v of values) count.set(v, (count.get(v) || 0) + 1);

#### cons
t keys = [...count.keys()].sort((a, b) => a - b);
    for (const start of keys) {

#### cons
t base = count.get(start);
        if (base > 0) {
            for (let k = 0; k < runLength; ++k) {

#### cons
t v = start + k;

#### cons
t have = count.get(v) || 0;
                if (have < base) return false;
                count.set(v, have - base);
            }
        }
    }
    return true;
}
Java
JAVA
import java.util.TreeMap;

class Solution {
    public boolean canPartitionRuns(int[] values, int runLength) {
        if (values.length % runLength != 0) return false;

        TreeMap<Integer, Integer> count = new TreeMap<>();
        for (int v : values) count.merge(v, 1, Integer::sum);

        // ascending iteration: the smallest live value anchors the next runs
        for (int start : count.keySet()) {
            int base = count.getOrDefault(start, 0);
            if (base > 0) {
                for (int k = 0; k < runLength; ++k) {
                    int v = start + k;
                    int have = count.getOrDefault(v, 0);
                    if (have < base) return false;
                    count.put(v, have - base);
                }
            }
        }
        return true;
    }
}
C++
CPP
#include <map>
#include <vector>
using namespace std;

class Solution {
public:
    bool canPartitionRuns(vector<int>& values, int runLength) {
        if (values.size() % runLength != 0) return false;

        map<int, long> count;            // ordered by value
        for (int v : values) ++count[v];

        // process the smallest remaining value first; it must start some runs
        for (const auto& [start, _] : count) {
            long base = count[start];
            if (base > 0) {
                for (int k = 0; k < runLength; ++k) {
                    int v = start + k;
                    if (count[v] < base) return false;   // not enough consecutive items
                    count[v] -= base;
                }
            }
        }
        return true;
    }
};
C
C
#include <stdbool.h>
#include <stdlib.h>

static int cmpInt(const void* a, const void* b) {
    int x = *(const int*)a, y = *(const int*)b;
    return (x > y) - (x < y);
}

bool canPartitionRuns(int* values, int valuesSize, int runLength) {
    if (valuesSize % runLength != 0) return false;

    qsort(values, valuesSize, sizeof(int), cmpInt);

    /* compress the sorted array into distinct values + their counts */
    int*  uniq = (int*)malloc(valuesSize * sizeof(int));
    long* cnt  = (long*)malloc(valuesSize * sizeof(long));
    int m = 0;
    for (int i = 0; i < valuesSize; ++i) {
        if (m > 0 && uniq[m - 1] == values[i]) {
            cnt[m - 1]++;
        } else {
            uniq[m] = values[i];
            cnt[m]  = 1;
            ++m;
        }
    }

    bool ok = true;
    for (int i = 0; i < m && ok; ++i) {
        if (cnt[i] == 0) continue;          /* already consumed by earlier runs */
        long base = cnt[i];                 /* this many runs must start at uniq[i] */
        for (int k = 0; k < runLength; ++k) {
            int j = i + k;                  /* the next consecutive value sits at i+k */
            if (j >= m || uniq[j] != uniq[i] + k || cnt[j] < base) {
                ok = false;
                break;
            }
            cnt[j] -= base;
        }
    }

    free(uniq);
    free(cnt);
    return ok;
}

#### Follow-up
questions
Return the actual groups (the list of runs) instead of just a boolean when a partition exists.
Handle a streaming variant where values arrive one at a time and you must report feasibility incrementally.
Replace the ordered map with a min-heap of distinct values plus a hash count — analyze how the complexity changes.
What if a group only needs consecutive values but the group size may vary between a low and high bound rather than being fixed?

