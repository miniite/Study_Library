---
type: "practice"
difficulty: "Other"
---

# Count Free Slots in an Operating Window

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A delivery robot is scheduled to operate for a fixed run of consecutive time slots numbered from 1 to totalSlots. During this run some slots are reserved by maintenance jobs.

You are given the integer totalSlots and a list reservations, where each entry reservations[i] = [from_i, to_i] means that every slot from from_i up to and including to_i is reserved by job i. Reservations may overlap one another, and every reserved slot lies inside the operating window.

Return the number of slots in the range [1, totalSlots] that are free — that is, slots during which the robot is available but no reservation covers them.

Input

An integer totalSlots.
A 2D integer array reservations of size n, each element a pair [from_i, to_i].

Output

A single integer: the count of free slots.

#### Cons
traints

1 <= totalSlots <= 10^9
1 <= reservations.length <= 10^5
reservations[i].length == 2
1 <= from_i <= to_i <= totalSlots

#### Example
s & test cases
TEXT
Input: totalSlots = 12, reservations = [[6,8],[1,4],[11,12]] -> Output: 3
Why: Reserved slots are {1,2,3,4,6,7,8,11,12}; the free slots are {5,9,10}, so 3. (Normal case with gaps between blocks.)

Input: totalSlots = 5, reservations = [[2,4],[1,3]] -> Output: 1
Why: The two intervals overlap and together cover slots 1..4; only slot 5 is free. (Overlapping intervals must not be double-counted.)

Input: totalSlots = 6, reservations = [[1,6]] -> Output: 0
Why: A single reservation covers the entire window, leaving no free slot. (Full coverage edge case.)

Input: totalSlots = 8, reservations = [[3,3]] -> Output: 7
Why: Only slot 3 is reserved; the other 7 slots are free. (Single-slot reservation plus leading and trailing gaps.)

Input: totalSlots = 10, reservations = [[2,3],[2,3],[2,3]] -> Output: 8
Why: Identical duplicate intervals cover only slots 2 and 3; 8 slots remain free. (Duplicates handled by merging.)

Input: totalSlots = 1, reservations = [[1,1]] -> Output: 0
Why: The lone slot is reserved. (Minimum-size window.)

#### Approach

Sort the intervals by start. Sweep left to right tracking prevEnd, the largest slot covered so far. For each interval, any gap between prevEnd + 1 and the interval's start is free and is added to the count; then extend prevEnd to the max of itself and the interval's end (so overlapping/duplicate intervals never double-count). After the loop, the tail from prevEnd + 1 to totalSlots is also free. This is the classic interval-merge / sweep-line pattern; counting gaps directly avoids materializing the up-to-10^9 slots.

Complexity

Time: O(n log n) — dominated by sorting the n reservations; the single sweep afterward is O(n). Space: O(1) auxiliary beyond the sort (O(n) if the sort is not in place). The answer never enumerates individual slots, so it is independent of totalSlots (which can be up to 10^9).

Python
PYTHON
from typing import List

def count_free_slots(total_slots: int, reservations: List[List[int]]) -> int:
    reservations.sort()
    prev_end = 0
    free = 0
    for start, end in reservations:
        if start > prev_end + 1:
            free += start - prev_end - 1
        prev_end = max(prev_end, end)
    free += total_slots - prev_end
    return free
JavaScript
JAVASCRIPT
function countFreeSlots(totalSlots, reservations) {
    reservations.sort((a, b) => a[0] - b[0]);
    let prevEnd = 0, free = 0;
    for (const [start, end] of reservations) {
        if (start > prevEnd + 1) {
            free += start - prevEnd - 1;
        }
        prevEnd = Math.max(prevEnd, end);
    }
    free += totalSlots - prevEnd;
    return free;
}
Java
JAVA
import java.util.Arrays;

class Solution {
    public int countFreeSlots(int totalSlots, int[][] reservations) {
        Arrays.sort(reservations, (a, b) -> Integer.compare(a[0], b[0]));
        long prevEnd = 0, freeSlots = 0;
        for (int[] r : reservations) {
            if (r[0] > prevEnd + 1)
                freeSlots += r[0] - prevEnd - 1;
            prevEnd = Math.max(prevEnd, r[1]);
        }
        freeSlots += totalSlots - prevEnd;
        return (int) freeSlots;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int countFreeSlots(int totalSlots, vector<vector<int>>& reservations) {
        sort(reservations.begin(), reservations.end());
        long long prevEnd = 0, freeSlots = 0;
        for (auto& r : reservations) {
            if (r[0] > prevEnd + 1)
                freeSlots += r[0] - prevEnd - 1;
            prevEnd = max(prevEnd, (long long)r[1]);
        }
        freeSlots += totalSlots - prevEnd;
        return (int)freeSlots;
    }
};
C
C
#include <stdlib.h>

static int cmp(const void* a, const void* b) {
    int* x = *(int**)a;
    int* y = *(int**)b;
    if (x[0] != y[0]) return x[0] - y[0];
    return x[1] - y[1];
}

int countFreeSlots(int totalSlots, int** reservations, int reservationsSize, int* reservationsColSize) {
    qsort(reservations, reservationsSize, sizeof(int*), cmp);
    long long prevEnd = 0, freeDays = 0;
    for (int i = 0; i < reservationsSize; i++) {
        int start = reservations[i][0], end = reservations[i][1];
        if (start > prevEnd + 1)
            freeDays += start - prevEnd - 1;
        if (end > prevEnd)
            prevEnd = end;
    }
    freeDays += totalSlots - prevEnd;
    return (int)freeDays;
}

#### Follow-up
questions
Instead of a count, return the list of maximal free intervals themselves.
What if reservations arrive as a stream and you must answer the free count after each insertion? (Consider a balanced BST / interval tree.)
How would you also report the total number of reserved slots without recomputing from scratch?
If totalSlots were astronomically large but reservation endpoints were sparse, does the approach still hold? (Yes — cost depends only on n.)

