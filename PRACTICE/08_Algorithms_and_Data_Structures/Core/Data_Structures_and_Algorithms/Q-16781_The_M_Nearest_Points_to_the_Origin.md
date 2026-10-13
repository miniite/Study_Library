---
type: "practice"
difficulty: "Other"
---

# The M Nearest Points to the Origin

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Heap (Priority Queue)](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heap_(Priority_Queue).md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array locations, where locations[i] = [xi, yi] is a point in the two-dimensional plane, and an integer m.

Return the m points that are closest to the origin (0, 0), where closeness is measured by ordinary straight-line (Euclidean) distance:

CODE
distance((x1, y1), (x2, y2)) = sqrt((x1 - x2)^2 + (y1 - y2)^2)

so the distance of a point (x, y) from the origin is sqrt(x^2 + y^2).

You may return the m points in any order. The set of the m nearest points is guaranteed to be unique (i.e. the answer is unique apart from ordering).

Note: because the square-root function is monotonic, you can compare x^2 + y^2 directly and never need to compute an actual square root.

Input: an array locations of [x, y] integer pairs, and an integer m. Output: an array of the m points nearest to the origin, in any order.

#### Cons
traints:

1 <= m <= locations.length <= 10^4
-10^4 <= xi, yi <= 10^4

#### Example
s & test cases
TEXT
Input: locations = [[2,4],[-1,1]], m = 1 -> Output: [[-1,1]]
Why: squared distances are 20 and 2; the single nearest point is [-1,1].

Input: locations = [[3,3],[5,-1],[-2,4]], m = 2 -> Output: [[3,3],[-2,4]]
Why: squared distances are 18, 26, 20; the two smallest are 18 and 20 (any order accepted).

Input: locations = [[1,0],[0,1],[2,2]], m = 3 -> Output: [[1,0],[0,1],[2,2]]
Why: m equals the number of points, so every point is returned.

Input: locations = [[0,0],[7,7]], m = 1 -> Output: [[0,0]]
Why: the origin point itself has distance 0 and is trivially the closest.

Input: locations = [[-2,-2],[1,1]], m = 1 -> Output: [[1,1]]
Why: squared distances are 8 and 2; negative coordinates are handled by squaring, so [1,1] wins.

#### Approach

Since square root is monotonic, rank points by squared distance x^2 + y^2 — no sqrt needed. Maintain a bounded max-heap of size m keyed by that squared distance: push each point, and whenever the heap grows past m, pop the current farthest. After scanning all points, the heap holds exactly the m nearest, which are returned in any order. Keeping the heap capped at m (rather than heapifying all n points) makes each operation O(log m). The underlying pattern is "top-k by a key using a size-k heap"; the same result can be reached in O(n) average time with quickselect (partition around a pivot distance until the m closest occupy the first m slots).

Complexity

Time: O(n log m), where n is the number of points — each point does at most one push/pop on a heap that never exceeds size m. Space: O(m) for the heap. Comparing squared distances avoids any floating-point work. (An average-case O(n) alternative exists via quickselect.)

Python
PYTHON
import heapq
from typing import List

class Solution:
    def kClosest(self, locations: List[List[int]], m: int) -> List[List[int]]:
        max_heap = []  # min-heap of (-dist, x, y) capped at size m
        for x, y in locations:
            d = x * x + y * y
            heapq.heappush(max_heap, (-d, x, y))
            if len(max_heap) > m:
                heapq.heappop(max_heap)
        return [[x, y] for _, x, y in max_heap]
JavaScript
JAVASCRIPT
function kClosest(locations, m) {
    // max-heap of size m keyed by squared distance; heap[i] = [dist, x, y]

#### cons
t heap = [];

#### cons
t dist = (x, y) => x * x + y * y;

#### cons
t up = (i) => {
        while (i > 0) {

#### cons
t p = (i - 1) >> 1;
            if (heap[p][0] < heap[i][0]) { [heap[p], heap[i]] = [heap[i], heap[p]]; i = p; }
            else break;
        }
    };

#### cons
t down = (i) => {

#### cons
t n = heap.length;
        while (true) {
            let l = 2 * i + 1, r = 2 * i + 2, big = i;
            if (l < n && heap[l][0] > heap[big][0]) big = l;
            if (r < n && heap[r][0] > heap[big][0]) big = r;
            if (big !== i) { [heap[i], heap[big]] = [heap[big], heap[i]]; i = big; }
            else break;
        }
    };

    for (const [x, y] of locations) {

#### cons
t d = dist(x, y);
        if (heap.length < m) { heap.push([d, x, y]); up(heap.length - 1); }
        else if (d < heap[0][0]) { heap[0] = [d, x, y]; down(0); }
    }
    return heap.map(([, x, y]) => [x, y]);
}
Java
JAVA
import java.util.PriorityQueue;

class Solution {
    public int[][] kClosest(int[][] locations, int m) {
        // max-heap of size m, keyed by squared distance
        PriorityQueue<int[]> maxHeap =
            new PriorityQueue<>((p, q) -> dist(q) - dist(p));
        for (int[] p : locations) {
            maxHeap.offer(p);
            if (maxHeap.size() > m) maxHeap.poll();
        }
        int[][] ans = new int[m][2];
        for (int i = 0; i < m; ++i) ans[i] = maxHeap.poll();
        return ans;
    }

    private int dist(int[] p) {
        return p[0] * p[0] + p[1] * p[1];
    }
}
C++
CPP
#include <vector>
#include <queue>
using namespace std;

class Solution {
public:
    vector<vector<int>> kClosest(vector<vector<int>>& locations, int m) {
        auto cmp = [] (const vector<int>& a, const vector<int>& b) {
            return (long long)a[0] * a[0] + (long long)a[1] * a[1] <
                   (long long)b[0] * b[0] + (long long)b[1] * b[1];
        };
        // max-heap keyed by squared distance, capped at size m
        priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> maxHeap(cmp);
        for (auto& p : locations) {
            maxHeap.push(p);
            if ((int)maxHeap.size() > m) maxHeap.pop();
        }
        vector<vector<int>> ans;
        while (!maxHeap.empty()) {
            ans.push_back(maxHeap.top());
            maxHeap.pop();
        }
        return ans;
    }
};
C
C
#include <stdlib.h>

typedef struct { long long dist; int x, y; } Node;

static void swapNode(Node* a, Node* b) { Node t = *a; *a = *b; *b = t; }

static void siftUp(Node* h, int i) {
    while (i > 0) {
        int p = (i - 1) / 2;
        if (h[p].dist < h[i].dist) { swapNode(&h[p], &h[i]); i = p; }
        else break;
    }
}

static void siftDown(Node* h, int n, int i) {
    while (1) {
        int l = 2 * i + 1, r = 2 * i + 2, big = i;
        if (l < n && h[l].dist > h[big].dist) big = l;
        if (r < n && h[r].dist > h[big].dist) big = r;
        if (big != i) { swapNode(&h[i], &h[big]); i = big; }
        else break;
    }
}

/* Returns m closest points; caller frees the result. */
int** kClosest(int** points, int pointsSize, int* pointsColSize, int m,
               int* returnSize, int** returnColumnSizes) {
    Node* heap = (Node*)malloc(sizeof(Node) * m);   /* max-heap of size m */
    int size = 0;
    for (int i = 0; i < pointsSize; ++i) {
        long long x = points[i][0], y = points[i][1];
        long long d = x * x + y * y;
        if (size < m) {
            heap[size].dist = d; heap[size].x = (int)x; heap[size].y = (int)y;
            siftUp(heap, size);
            ++size;
        } else if (d < heap[0].dist) {
            heap[0].dist = d; heap[0].x = (int)x; heap[0].y = (int)y;
            siftDown(heap, size, 0);
        }
    }
    int** ans = (int**)malloc(sizeof(int*) * m);
    *returnColumnSizes = (int*)malloc(sizeof(int) * m);
    for (int i = 0; i < m; ++i) {
        ans[i] = (int*)malloc(sizeof(int) * 2);
        ans[i][0] = heap[i].x;
        ans[i][1] = heap[i].y;
        (*returnColumnSizes)[i] = 2;
    }
    *returnSize = m;
    free(heap);
    return ans;
}

#### Follow-up
questions
The points arrive as an unbounded stream too large to store — how do you keep the m nearest? (The size-m max-heap already handles this.)
Return the m points sorted by increasing distance instead of in arbitrary order.
Achieve O(n) average time — describe the quickselect partitioning around a pivot distance.
If distances can tie exactly at the m-th position and the answer is no longer unique, how do you define and break ties deterministically?

