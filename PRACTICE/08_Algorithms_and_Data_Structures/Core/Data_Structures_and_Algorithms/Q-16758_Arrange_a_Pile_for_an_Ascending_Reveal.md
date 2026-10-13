---
type: "practice"
difficulty: "Other"
---

# Arrange a Pile for an Ascending Reveal

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queue.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array values of distinct integers. Think of them as a stack of face-down tiles that you may pre-arrange into any order you like before the game begins. The top of the stack is the first element of your arrangement.

Once the arrangement is fixed, tiles are turned up by repeating the following procedure until the stack is empty:

Take the tile currently on top of the stack, turn it face up (reveal it), and remove it.
If the stack is not yet empty, take the tile that is now on top and move it to the bottom of the stack.
If any tiles remain, go back to step 1.

Your job is to return an arrangement of values such that the tiles are revealed in strictly increasing order.

It is guaranteed that a valid arrangement always exists, and because all values are distinct, the arrangement is unique.

Input: an array values of distinct integers.

Output: an array — a permutation of values — whose first element is the top of the stack, such that following the reveal procedure above turns tiles up in increasing order.

#### Cons
traints:

1 <= values.length <= 1000
1 <= values[i] <= 10^6
All entries of values are distinct.

#### Example
s & test cases
TEXT
Input: values = [12, 9, 4, 21, 6] -> Output: [4, 21, 6, 12, 9]
Why: revealing follows 4 (move 21 down), 6 (move 12 down), 9 (move 21 down), 12 (move 21 down), 21 — the ascending sequence 4,6,9,12,21.

Input: values = [8, 3] -> Output: [3, 8]
Why: reveal 3, move 8 to the bottom, then reveal 8 — order 3,8.

Input: values = [5] -> Output: [5]
Why: a single tile is simply revealed; no rearrangement is possible or needed.

Input: values = [7, 2, 9] -> Output: [2, 9, 7]
Why: reveal 2 (move 9 down -> [7,9]), reveal 7 (move 9 down -> [9]), reveal 9 — order 2,7,9.

Input: values = [1000000, 1, 500000, 250000] -> Output: [1, 500000, 250000, 1000000]
Why: large and small values behave identically; the reveal order is 1, 250000, 500000, 1000000.

#### Approach

Reverse the game: instead of guessing an arrangement, decide which final position each value must occupy. The smallest value is revealed first, the next-smallest second, and so on, so sort the values ascending and walk them in that order. Simulate the reveal mechanics on the indices 0..n-1 rather than the values: keep a queue of the still-unassigned destination positions in their natural front-to-back order. Repeatedly pop the front position and assign it the next-smallest value (that position is "revealed" now), then, mirroring "move the next top card to the bottom", pop the new front position and push it to the back. When the queue empties, result[pos] = value has been filled for every position and is exactly the arrangement that reveals in increasing order. The key pattern is running the process forward on positions with a queue while feeding values in sorted order.

Complexity

Time: O(n log n), dominated by sorting the values; the simulation over the position queue does O(1) amortized work per step across O(n) steps. Space: O(n) for the sorted copy, the position queue, and the output array.

Python
PYTHON
from collections import deque
from typing import List

class Solution:
    def arrangeForAscendingReveal(self, values: List[int]) -> List[int]:
        n = len(values)
        order = sorted(values)
        positions = deque(range(n))          # queue of destination positions
        result = [0] * n
        for value in order:                  # smallest first
            pos = positions.popleft()
            result[pos] = value
            if positions:                    # move the next position to the bottom
                positions.append(positions.popleft())
        return result
JavaScript
JAVASCRIPT
function arrangeForAscendingReveal(values) {

#### cons
t n = values.length;

#### cons
t order = [...values].sort((a, b) => a - b);

    // queue of destination positions, using a head pointer instead of shift()

#### cons
t positions = [];
    for (let i = 0; i < n; ++i) positions.push(i);

#### cons
t result = new Array(n);
    let head = 0;
    for (let i = 0; i < n; ++i) {

#### cons
t pos = positions[head++];       // dequeue front position
        result[pos] = order[i];              // smallest remaining value lands here
        if (head < positions.length) {       // recycle next position to the back
            positions.push(positions[head++]);
        }
    }
    return result;
}
Java
JAVA
import java.util.*;

class Solution {
    public int[] arrangeForAscendingReveal(int[] values) {
        int n = values.length;
        int[] sorted = values.clone();
        Arrays.sort(sorted);

        Deque<Integer> q = new ArrayDeque<>();   // queue of destination positions
        for (int i = 0; i < n; ++i) q.addLast(i);

        int[] result = new int[n];
        for (int i = 0; i < n; ++i) {
            int pos = q.pollFirst();
            result[pos] = sorted[i];             // smallest remaining value goes here
            if (!q.isEmpty()) {                  // recycle the next position to the back
                q.addLast(q.pollFirst());
            }
        }
        return result;
    }
}
C++
CPP
#include <vector>
#include <deque>
#include <algorithm>
using namespace std;

class Solution {
public:
    vector<int> arrangeForAscendingReveal(vector<int>& values) {
        int n = values.size();
        vector<int> sorted(values);
        sort(sorted.begin(), sorted.end());

        deque<int> q;                       // queue of destination positions
        for (int i = 0; i < n; ++i) q.push_back(i);

        vector<int> result(n);
        for (int i = 0; i < n; ++i) {
            int pos = q.front(); q.pop_front();
            result[pos] = sorted[i];        // smallest remaining value lands here
            if (!q.empty()) {               // simulate "move next to the bottom"
                q.push_back(q.front());
                q.pop_front();
            }
        }
        return result;
    }
};
C
C
#include <stdlib.h>

static int cmpInt(const void* a, const void* b) {
    int x = *(const int*)a, y = *(const int*)b;
    return (x > y) - (x < y);
}

/* Returns a malloc'd array (caller frees). *returnSize is set to valuesSize. */
int* arrangeForAscendingReveal(int* values, int valuesSize, int* returnSize) {
    int n = valuesSize;
    *returnSize = n;
    int* result = (int*)malloc(sizeof(int) * (n > 0 ? n : 1));
    if (n == 0) return result;

    /* sort a copy so the input array is left untouched */
    int* sorted = (int*)malloc(sizeof(int) * n);
    for (int i = 0; i < n; ++i) sorted[i] = values[i];
    qsort(sorted, n, sizeof(int), cmpInt);

    /* circular queue of destination positions 0..n-1 */
    int* q = (int*)malloc(sizeof(int) * n);
    for (int i = 0; i < n; ++i) q[i] = i;
    int head = 0, count = n;

    for (int i = 0; i < n; ++i) {
        int pos = q[head];                 /* dequeue the position for this reveal */
        head = (head + 1) % n; count--;
        result[pos] = sorted[i];
        if (count > 0) {                   /* recycle the next position to the back */
            int nxt = q[head];
            head = (head + 1) % n; count--;
            q[(head + count) % n] = nxt;
            count++;
        }
    }
    free(sorted);
    free(q);
    return result;
}

#### Follow-up
questions
Given a fixed arrangement instead, output the order in which the tiles are actually revealed (the inverse mapping).
Generalize the rule so that after each reveal you move the next k tiles to the bottom rather than exactly one; return the arrangement for a sorted reveal.
Reconstruct the answer using only index arithmetic, without an explicit queue.
What changes if the values are allowed to repeat, and how do you break ties so the reveal is non-decreasing?

