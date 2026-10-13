---
type: "practice"
difficulty: "Other"
---

# Minimum Difference Between Two Halves After Dropping a Third

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Heap (Priority Queue)](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heap_(Priority_Queue).md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed integer array values whose length is exactly 3 * k for some positive integer k.

You must delete a subsequence of exactly k elements from values (you choose which k positions to drop; the relative order of the surviving elements is preserved). After the deletion, 2 * k elements remain. Split them, keeping their order, into two equal halves:

the first k surviving elements form the left half, with sum leftSum;
the next k surviving elements form the right half, with sum rightSum.

The difference of this arrangement is leftSum - rightSum (this value may be negative).

Over all legal choices of which k elements to delete, return the minimum possible value of leftSum - rightSum.

Because the survivors keep their original order, this is equivalent to picking some split boundary: the left half is drawn from a prefix of the array and the right half from the complementary suffix. To make the difference as small as possible you want the left half to be as small as possible and the right half to be as large as possible.

#### Cons
traints:

values.length == 3 * k
1 <= k <= 10^5
1 <= values[i] <= 10^5
The answer can be as small as roughly -10^{10}, so use 64-bit arithmetic for the running sums.

#### Example
s & test cases
TEXT
Input: values = [5, 2, 4] -> Output: -2
Why: k = 1, drop one element. Dropping 5 leaves [2,4] → 2 - 4 = -2 (best). Dropping 2 → 5 - 4 = 1; dropping 4 → 5 - 2 = 3. Minimum is -2.

Input: values = [2, 6, 4, 3, 1, 5] -> Output: -2
Why: k = 2. Drop index 1 (6) and index 4 (1): survivors [2,4,3,5] → (2+4) - (3+5) = 6 - 8 = -2, the smallest achievable.

Input: values = [1, 1, 1] -> Output: 0
Why: k = 1 with all duplicates. Any deletion leaves [1,1] → 1 - 1 = 0. Edge: identical elements.

Input: values = [10, 1, 1, 1, 1, 10] -> Output: -9
Why: k = 2. Keep the two small 1's on the left and the large 10 (plus a 1) on the right: (1+1) - (10+1) = 2 - 11 = -9. Extreme values pushed to opposite halves.

Input: values = [6, 6, 6, 6, 6, 6] -> Output: 0
Why: k = 2, every element equal, so every valid arrangement yields sum difference 0. Edge: uniform array.

#### Approach

Since the survivors preserve order, the left half is always drawn from some prefix and the right half from the complementary suffix, meeting at a boundary. Fix a boundary index i between position n-1 and 2n-1: the left half takes k elements from values[0..i] and the right half takes k from values[i+1..end]. To minimize leftSum - rightSum at that boundary you independently minimize the left sum and maximize the right sum. Precompute both with heaps: sweeping left to right, a max-heap of size n keeps the n smallest prefix elements, giving leftMin[i] for every prefix; sweeping right to left, a min-heap of size n keeps the n largest suffix elements, giving rightMax[i] for every suffix. The answer is min over i of leftMin[i] - rightMax[i+1]. This is the classic prefix-min / suffix-max with a bounded heap pattern (a greedy "keep the best n so far" running selection), which turns an exponential subset choice into two linear-ish passes.

Complexity

Time O(n log n), Space O(n), where n = k = len/3. Each of the two sweeps pushes/pops every element through a size-n heap once (O(log n) per operation), and the final boundary scan is O(n). The two prefix/suffix arrays and the heap use O(n) memory.

Python
PYTHON
import heapq
from typing import List

class Solution:
    def minHalvesDifference(self, values: List[int]) -> int:
        length = len(values)
        n = length // 3

        # left_min[i] = minimum sum of n elements chosen from values[0..i]
        left_min = [0] * length
        max_heap = []          # store negatives to emulate a max-heap
        s = 0
        for i in range(n):
            s += values[i]
            heapq.heappush(max_heap, -values[i])
        left_min[n - 1] = s
        for i in range(n, 2 * n):
            s += values[i]
            heapq.heappush(max_heap, -values[i])
            s -= -heapq.heappop(max_heap)      # drop largest -> keep n smallest
            left_min[i] = s

        # right_max[i] = maximum sum of n elements chosen from values[i..end]
        right_max = [0] * (length + 1)
        min_heap = []
        s = 0
        for i in range(length - 1, length - n - 1, -1):
            s += values[i]
            heapq.heappush(min_heap, values[i])
        right_max[2 * n] = s
        for i in range(2 * n - 1, n - 1, -1):
            s += values[i]
            heapq.heappush(min_heap, values[i])
            s -= heapq.heappop(min_heap)        # drop smallest -> keep n largest
            right_max[i] = s

        return min(left_min[i] - right_max[i + 1] for i in range(n - 1, 2 * n))
JavaScript
JAVASCRIPT
class Heap {

#### cons
tructor(cmp) { this.a = []; this.cmp = cmp; }
    size() { return this.a.length; }
    push(v) {

#### cons
t a = this.a; a.push(v); let i = a.length - 1;
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (this.cmp(a[i], a[p]) < 0) { [a[i], a[p]] = [a[p], a[i]]; i = p; }
            else break;
        }
    }
    pop() {

#### cons
t a = this.a, top = a[0], last = a.pop();
        if (a.length) {
            a[0] = last; let i = 0; const n = a.length;
            while (true) {
                let l = 2 * i + 1, r = 2 * i + 2, b = i;
                if (l < n && this.cmp(a[l], a[b]) < 0) b = l;
                if (r < n && this.cmp(a[r], a[b]) < 0) b = r;
                if (b === i) break;
                [a[i], a[b]] = [a[b], a[i]]; i = b;
            }
        }
        return top;
    }
}

function minHalvesDifference(values) {

#### cons
t len = values.length, n = len / 3;

#### cons
t leftMin = new Array(len).fill(0);

#### cons
t rightMax = new Array(len + 1).fill(0);

#### cons
t maxHeap = new Heap((x, y) => y - x);   // largest on top -> keep n smallest
    let s = 0;
    for (let i = 0; i < n; i++) { s += values[i]; maxHeap.push(values[i]); }
    leftMin[n - 1] = s;
    for (let i = n; i < 2 * n; i++) {
        s += values[i]; maxHeap.push(values[i]);
        s -= maxHeap.pop();
        leftMin[i] = s;
    }

#### cons
t minHeap = new Heap((x, y) => x - y);   // smallest on top -> keep n largest
    s = 0;
    for (let i = len - 1; i >= len - n; i--) { s += values[i]; minHeap.push(values[i]); }
    rightMax[2 * n] = s;
    for (let i = 2 * n - 1; i >= n; i--) {
        s += values[i]; minHeap.push(values[i]);
        s -= minHeap.pop();
        rightMax[i] = s;
    }

    let ans = Infinity;
    for (let i = n - 1; i < 2 * n; i++)
        ans = Math.min(ans, leftMin[i] - rightMax[i + 1]);
    return ans;
}
Java
JAVA
import java.util.PriorityQueue;
import java.util.Collections;

class Solution {
    public long minHalvesDifference(int[] values) {
        int len = values.length, n = len / 3;
        long[] leftMin  = new long[len];      // min sum of n taken from prefix ending at i
        long[] rightMax = new long[len + 1];  // max sum of n taken from suffix starting at i

        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        long s = 0;
        for (int i = 0; i < n; i++) { s += values[i]; maxHeap.offer(values[i]); }
        leftMin[n - 1] = s;
        for (int i = n; i < 2 * n; i++) {
            s += values[i]; maxHeap.offer(values[i]);
            s -= maxHeap.poll();              // drop the largest -> keep n smallest
            leftMin[i] = s;
        }

        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        s = 0;
        for (int i = len - 1; i >= len - n; i--) { s += values[i]; minHeap.offer(values[i]); }
        rightMax[2 * n] = s;
        for (int i = 2 * n - 1; i >= n; i--) {
            s += values[i]; minHeap.offer(values[i]);
            s -= minHeap.poll();              // drop the smallest -> keep n largest
            rightMax[i] = s;
        }

        long ans = Long.MAX_VALUE;
        for (int i = n - 1; i < 2 * n; i++)
            ans = Math.min(ans, leftMin[i] - rightMax[i + 1]);
        return ans;
    }
}
C++
CPP
#include <vector>
#include <queue>
#include <climits>
#include <algorithm>
using namespace std;

class Solution {
public:
    long long minHalvesDifference(vector<int>& values) {
        int len = values.size(), n = len / 3;
        vector<long long> leftMin(len), rightMax(len + 1);

        priority_queue<int> maxHeap;          // keeps the n smallest of a prefix
        long long s = 0;
        for (int i = 0; i < n; i++) { s += values[i]; maxHeap.push(values[i]); }
        leftMin[n - 1] = s;
        for (int i = n; i < 2 * n; i++) {
            s += values[i]; maxHeap.push(values[i]);
            s -= maxHeap.top(); maxHeap.pop();
            leftMin[i] = s;
        }

        priority_queue<int, vector<int>, greater<int>> minHeap; // keeps the n largest of a suffix
        s = 0;
        for (int i = len - 1; i >= len - n; i--) { s += values[i]; minHeap.push(values[i]); }
        rightMax[2 * n] = s;
        for (int i = 2 * n - 1; i >= n; i--) {
            s += values[i]; minHeap.push(values[i]);
            s -= minHeap.top(); minHeap.pop();
            rightMax[i] = s;
        }

        long long ans = LLONG_MAX;
        for (int i = n - 1; i < 2 * n; i++)
            ans = min(ans, leftMin[i] - rightMax[i + 1]);
        return ans;
    }
};
C
C
#include <stdlib.h>
#include <limits.h>

/* isMin==0 -> max-heap; isMin==1 -> min-heap, over long long */
static void hpush(long long *a, int *sz, long long v, int isMin) {
    int i = (*sz)++;
    a[i] = v;
    while (i > 0) {
        int p = (i - 1) / 2;
        int swap = isMin ? (a[p] > a[i]) : (a[p] < a[i]);
        if (!swap) break;
        long long t = a[p]; a[p] = a[i]; a[i] = t;
        i = p;
    }
}
static long long hpop(long long *a, int *sz, int isMin) {
    long long top = a[0];
    a[0] = a[--(*sz)];
    int i = 0;
    while (1) {
        int l = 2 * i + 1, r = 2 * i + 2, b = i;
        if (l < *sz && (isMin ? a[l] < a[b] : a[l] > a[b])) b = l;
        if (r < *sz && (isMin ? a[r] < a[b] : a[r] > a[b])) b = r;
        if (b == i) break;
        long long t = a[b]; a[b] = a[i]; a[i] = t;
        i = b;
    }
    return top;
}

long long minHalvesDifference(int *values, int len) {
    int n = len / 3;
    long long *leftMin  = malloc(sizeof(long long) * len);       /* min sum of n from prefix[0..i] */
    long long *rightMax = malloc(sizeof(long long) * (len + 1)); /* max sum of n from suffix[i..end] */
    long long *heap     = malloc(sizeof(long long) * (len + 1));
    int hs = 0;
    long long s = 0;

    for (int i = 0; i < n; i++) { s += values[i]; hpush(heap, &hs, values[i], 0); }
    leftMin[n - 1] = s;
    for (int i = n; i < 2 * n; i++) {
        s += values[i]; hpush(heap, &hs, values[i], 0);
        s -= hpop(heap, &hs, 0);          /* drop the largest -> keep n smallest */
        leftMin[i] = s;
    }

    hs = 0; s = 0;
    for (int i = len - 1; i >= len - n; i--) { s += values[i]; hpush(heap, &hs, values[i], 1); }
    rightMax[2 * n] = s;
    for (int i = 2 * n - 1; i >= n; i--) {
        s += values[i]; hpush(heap, &hs, values[i], 1);
        s -= hpop(heap, &hs, 1);          /* drop the smallest -> keep n largest */
        rightMax[i] = s;
    }

    long long ans = LLONG_MAX;
    for (int i = n - 1; i < 2 * n; i++) {
        long long d = leftMin[i] - rightMax[i + 1];
        if (d < ans) ans = d;
    }
    free(leftMin); free(rightMax); free(heap);
    return ans;
}

#### Follow-up
questions
What changes if the two halves must have unequal sizes (say the left keeps p elements and the right keeps q, p + q + removed = length)? How do the boundary bounds shift?
Can you reconstruct which k indices were actually removed, not just the optimal difference value?
If values could be negative, does the prefix-min / suffix-max heap argument still hold? Explain why or why not.
How would you answer many independent queries that each restrict the boundary to a subrange, faster than recomputing per query?

