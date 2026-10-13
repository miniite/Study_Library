---
type: "practice"
difficulty: "Other"
---

# Reorder Values by Ascending Frequency

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array of integers values. Rearrange the elements so that the array is ordered by how often each value occurs, from the least frequent to the most frequent. When two different values occur the same number of times, the one with the larger numeric value must come first (that is, ties are broken by value in decreasing order).

All copies of a given value stay together as a contiguous block; only the relative order of the blocks (and the tie-breaking order among equally frequent blocks) is determined by the rules above. Return the rearranged array.

#### Cons
traints

1 <= values.length <= 100
-100 <= values[i] <= 100

Clarifications

Frequency is the number of times a value appears in the whole array.
A value that appears once is "less frequent" than one that appears twice, so it is placed earlier.
If several values share a frequency, list them in strictly decreasing numeric order among themselves.

#### Example
s & test cases
TEXT
Input: values = [4,4,6,6,6,9] -> Output: [9,4,4,6,6,6]
Why: 9 occurs once, 4 occurs twice, 6 occurs three times, so blocks go least-to-most frequent.

Input: values = [7,8,5,8,7] -> Output: [5,8,8,7,7]
Why: 5 occurs once and comes first; 7 and 8 both occur twice, so the tie is broken by larger value first (8 before 7).

Input: values = [-2,3,-7,1,2,-7,3,1,3] -> Output: [2,-2,1,1,-7,-7,3,3,3]
Why: frequency-1 values are 2 and -2 (larger first), frequency-2 values are 1 and -7 (larger first), then 3 occurs three times.

Input: values = [5] -> Output: [5]
Why: a single element trivially has frequency 1 and is returned unchanged.

Input: values = [10,10,20,20] -> Output: [20,20,10,10]
Why: both values occur twice, so the tie is resolved by placing the larger value (20) before the smaller (10).

Input: values = [-100,-100,100] -> Output: [100,-100,-100]
Why: 100 occurs once (placed first), -100 occurs twice; tests the extreme bounds of the value range.

#### Approach

Count how many times each value appears using a hash map (or, since the value range is tiny and fixed, a direct-indexed counter array of size 201). Then sort the elements with a two-key comparator: the primary key is the value's frequency in ascending order, and the secondary key (for equal frequencies) is the value itself in descending order. In Python this is expressed cleanly as the sort key (freq[v], -v). Every element compares by the frequency of its own value, so identical values naturally cluster into contiguous blocks. This is a standard "sort by a derived key with a tie-breaker" pattern.

Complexity

Time: O(n log n) — building the frequency map is O(n), and the comparison sort of n elements dominates at O(n log n). Space: O(k) for the frequency map where k is the number of distinct values (at most 201 here), plus O(n) if a boxed/copy array is used for sorting.

Python
PYTHON
from collections import Counter
from typing import List

class Solution:
    def frequencySort(self, values: List[int]) -> List[int]:
        freq = Counter(values)
        # Sort key: ascending frequency, then descending value for ties.
        return sorted(values, key=lambda v: (freq[v], -v))
JavaScript
JAVASCRIPT
function frequencySort(values) {

#### cons
t freq = new Map();
    for (const v of values) freq.set(v, (freq.get(v) || 0) + 1);

    return values
        .slice()
        .sort((a, b) => (freq.get(a) - freq.get(b)) || (b - a));
}
Java
JAVA
import java.util.*;

class Solution {
    public int[] frequencySort(int[] values) {
        Map<Integer, Integer> freq = new HashMap<>();
        for (int v : values) freq.merge(v, 1, Integer::sum);

        Integer[] boxed = new Integer[values.length];
        for (int i = 0; i < values.length; i++) boxed[i] = values[i];

        Arrays.sort(boxed, (a, b) -> {
            if (!freq.get(a).equals(freq.get(b))) return freq.get(a) - freq.get(b); // ascending freq
            return b - a;                                                            // tie: descending value
        });

        int[] result = new int[values.length];
        for (int i = 0; i < values.length; i++) result[i] = boxed[i];
        return result;
    }
}
C++
CPP
#include <vector>
#include <unordered_map>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<int> frequencySort(vector<int>& values) {
        unordered_map<int, int> freq;
        for (int v : values) freq[v]++;
        sort(values.begin(), values.end(), [&] (int a, int b) {
            if (freq[a] != freq[b]) return freq[a] < freq[b];  // ascending frequency
            return a > b;                                       // tie: descending value
        });
        return values;
    }
};
C
C
#include <stdlib.h>

/* Value range is [-100, 100]; shift by 100 to index a fixed table. */
static int gFreq[201];

static int cmp(const void *a, const void *b) {
    int x = *(const int *)a;
    int y = *(const int *)b;
    int fx = gFreq[x + 100];
    int fy = gFreq[y + 100];
    if (fx != fy) return fx - fy;   /* ascending frequency */
    return y - x;                   /* tie: descending value */
}

int *frequencySort(int *values, int valuesSize, int *returnSize) {
    for (int i = 0; i < 201; i++) gFreq[i] = 0;
    for (int i = 0; i < valuesSize; i++) gFreq[values[i] + 100]++;
    qsort(values, valuesSize, sizeof(int), cmp);
    *returnSize = valuesSize;
    return values;
}

#### Follow-up
questions
How would the comparator change if ties should instead be broken by increasing value?
If the values could span the full 32-bit integer range instead of [-100, 100], would you still use a counter array, and why or why not?
Sort by frequency in descending order instead — what part of the key flips?
Return only the k least frequently occurring values; can you avoid a full sort with a heap or bucketing?

