---
type: "practice"
difficulty: "Other"
---

# Markers on the Tightest Enclosing Boundary

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Geometry](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Geometry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list points, where points[i] = [xi, yi] are the integer coordinates of the i-th marker driven into a flat field.

You want to run a single closed loop of fencing around the markers so that every marker is either strictly inside the loop or lying exactly on it, while spending the least total length of fence possible. Pulling the fence tight around the outermost markers minimizes its length, so the loop traces the convex boundary of the point set.

A marker is said to lie on the fence if it sits on this boundary. Importantly, this includes markers that fall exactly on a straight side of the loop between two corner markers — not only the corner markers themselves. Such a marker is touched by the taut fence, so it counts as being on the boundary.

Return the coordinates of every marker that lies on the fence. You may return them in any order.

Input
points: a list of [x, y] integer coordinate pairs. All pairs are distinct.
Output
A list of the coordinate pairs that lie on the boundary of the tightest enclosing loop, in any order.
Notes / special cases
If there is a single marker, it is trivially on the boundary and is returned by itself.
If all markers are collinear, the "loop" degenerates to a segment that runs through all of them, so every marker lies on it and all are returned.
Because the boundary must include markers lying on the interior of an edge, this is slightly different from a standard convex hull that keeps only corner vertices — here every point on the hull's perimeter is part of the answer.

#### Cons
traints
1 <= points.length <= 3000
points[i].length == 2
0 <= xi, yi <= 100
All coordinates are distinct.

#### Example
s & test cases
TEXT
Input: points = [[0,0],[2,0],[4,0],[0,4],[4,4],[2,2]] -> Output: [[0,0],[2,0],[4,0],[4,4],[0,4]]
Why: The tight loop is the 4x4 square through the four corners. [2,2] is strictly inside and is excluded, while [2,0] lies exactly on the bottom edge between [0,0] and [4,0], so it is kept even though it is not a corner. (Collinear-on-edge case.)

Input: points = [[0,0],[4,4],[8,8],[8,0]] -> Output: [[0,0],[4,4],[8,0],[8,8]]
Why: [4,4] sits on the diagonal side from [0,0] to [8,8], so it is a boundary point on a slanted edge and is returned along with the three corners. (Collinear on a non-axis edge.)

Input: points = [[0,1],[1,1],[3,1]] -> Output: [[0,1],[1,1],[3,1]]
Why: All three markers are collinear, so the fence collapses to a horizontal segment passing through every marker; all of them lie on it. (Degenerate / all-collinear case.)

Input: points = [[2,3]] -> Output: [[2,3]]
Why: A single marker is trivially on the boundary. (Minimum-size edge case.)

Input: points = [[0,0],[5,0],[5,5],[0,5],[3,3]] -> Output: [[0,0],[5,0],[5,5],[0,5]]
Why: The four corners form the square; [3,3] is strictly interior and is dropped. (Interior point excluded, no collinear ties.)

Input: points = [[0,0],[1,0],[0,1],[1,1]] -> Output: [[0,0],[1,0],[0,1],[1,1]]
Why: Every marker is a corner of the unit square, so all four are on the perimeter. (All points on the hull.)

#### Approach

The tightest enclosing loop is the convex hull of the point set, so the task reduces to reporting every point that lies on the hull's perimeter — including points sitting on the interior of an edge, which a plain hull would drop. We use Andrew's monotone chain: sort the points by x (ties by y), then sweep left-to-right building a lower chain and right-to-left building an upper chain. The pivotal detail versus a standard hull is the pop condition: we discard the previous point only when the new triple makes a strict turn (cross > 0). When three points are collinear (cross == 0) we keep the middle one, which is exactly what makes points lying flat on an edge appear in the answer. Because the lower and upper chains can share the same boundary points (and collinear runs can be re-added by both sweeps), we finish with a dedup (a set, or sort+unique). The cross product (q.y-p.y)*(r.x-q.x) - (q.x-p.x)*(r.y-q.y) is computed with 64-bit intermediates to avoid overflow. Pattern: computational geometry / convex hull with collinear-boundary retention.

Complexity

Time: O(n log n), where n is the number of markers — dominated by the coordinate sort; each of the two hull passes and the final dedup is O(n) (every point is pushed/popped a constant number of times). Space: O(n) for the sorted copy and the hull stack.

Python
PYTHON
from typing import List

class Solution:
    def boundaryPoints(self, points: List[List[int]]) -> List[List[int]]:
        if len(points) <= 1:
            return [list(p) for p in points]

        pts = sorted(points, key=lambda p: (p[0], p[1]))

        def cross(p, q, r):
            # Sign of the turn made by the top two hull points and r.
            return (q[1] - p[1]) * (r[0] - q[0]) - (q[0] - p[0]) * (r[1] - q[1])

        hull = []
        # Lower pass, left to right; pop only on a strict turn so edge points stay.
        for p in pts:
            while len(hull) > 1 and cross(hull[-1], hull[-2], p) > 0:
                hull.pop()
            hull.append((p[0], p[1]))
        hull.pop()

        # Upper pass, right to left.
        for p in reversed(pts):
            while len(hull) > 1 and cross(hull[-1], hull[-2], p) > 0:
                hull.pop()
            hull.append((p[0], p[1]))

        # Both passes meet on shared boundary points; a set removes duplicates.
        return [list(p) for p in set(hull)]
JavaScript
JAVASCRIPT
/**
 * @param {number[][]} points
 * @return {number[][]}
 */
var boundaryPoints = function (points) {
    if (points.length <= 1) return points.map(p => [p[0], p[1]]);

#### cons
t pts = points.slice().sort((a, b) => (a[0] === b[0] ? a[1] - b[1] : a[0] - b[0]));

    // Sign of the turn made by the top two hull points and r.

#### cons
t cross = (p, q, r) =>
        (q[1] - p[1]) * (r[0] - q[0]) - (q[0] - p[0]) * (r[1] - q[1]);

#### cons
t hull = [];

    // Lower pass, left to right; discard only strict turns so collinear edge points survive.
    for (const p of pts) {
        while (hull.length > 1 &&
               cross(hull[hull.length - 1], hull[hull.length - 2], p) > 0) hull.pop();
        hull.push(p);
    }
    hull.pop();

    // Upper pass, right to left.
    for (let i = pts.length - 1; i >= 0; i--) {

#### cons
t p = pts[i];
        while (hull.length > 1 &&
               cross(hull[hull.length - 1], hull[hull.length - 2], p) > 0) hull.pop();
        hull.push(p);
    }

    // The two passes meet on shared endpoints; dedup with a coordinate key.

#### cons
t seen = new Set();

#### cons
t res = [];
    for (const p of hull) {

#### cons
t key = p[0] + ',' + p[1];
        if (!seen.has(key)) { seen.add(key); res.push([p[0], p[1]]); }
    }
    return res;
};
Java
JAVA
import java.util.*;

class Solution {
    // Cross-product sign of the turn made by the top two hull points and r.
    private static long cross(int[] p, int[] q, int[] r) {
        return (long)(q[1] - p[1]) * (r[0] - q[0]) - (long)(q[0] - p[0]) * (r[1] - q[1]);
    }

    public int[][] boundaryPoints(int[][] points) {
        if (points.length <= 1) return points;

        int[][] pts = points.clone();
        Arrays.sort(pts, (a, b) -> a[0] == b[0] ? a[1] - b[1] : a[0] - b[0]);

        int[][] stack = new int[2 * pts.length + 1][];
        int top = 0;

        // Lower pass, left to right; keep collinear points by discarding only strict turns.
        for (int[] p : pts) {
            while (top > 1 && cross(stack[top - 1], stack[top - 2], p) > 0) top--;
            stack[top++] = p;
        }
        top--;

        // Upper pass, right to left.
        for (int i = pts.length - 1; i >= 0; i--) {
            while (top > 1 && cross(stack[top - 1], stack[top - 2], pts[i]) > 0) top--;
            stack[top++] = pts[i];
        }

        // The two passes share endpoints; dedup by an integer key (coords fit in 0..100).
        Set<Long> seen = new HashSet<>();
        List<int[]> res = new ArrayList<>();
        for (int i = 0; i < top; i++) {
            long key = (long) stack[i][0] * 1000003L + stack[i][1];
            if (seen.add(key)) res.add(stack[i]);
        }
        return res.toArray(new int[0][]);
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
    // Cross-product sign of the turn formed by the top two hull points and r.
    static long cross(const vector<int>& p, const vector<int>& q, const vector<int>& r) {
        return (long)(q[1] - p[1]) * (r[0] - q[0]) - (long)(q[0] - p[0]) * (r[1] - q[1]);
    }

public:
    vector<vector<int>> boundaryPoints(vector<vector<int>> points) {
        if (points.size() <= 1) return points;

        sort(points.begin(), points.end(), [] (const vector<int>& a, const vector<int>& b) {
            return a[0] == b[0] ? a[1] < b[1] : a[0] < b[0];
        });

        vector<vector<int>> hull;

        // Lower pass, left to right; pop only on a strict turn so edge points survive.
        for (const auto& p : points) {
            while (hull.size() > 1 && cross(hull[hull.size() - 1], hull[hull.size() - 2], p) > 0)
                hull.pop_back();
            hull.push_back(p);
        }
        hull.pop_back();

        // Upper pass, right to left.
        for (int i = (int)points.size() - 1; i >= 0; --i) {
            while (hull.size() > 1 && cross(hull[hull.size() - 1], hull[hull.size() - 2], points[i]) > 0)
                hull.pop_back();
            hull.push_back(points[i]);
        }

        // Remove points that appear in both passes.
        sort(hull.begin(), hull.end());
        hull.erase(unique(hull.begin(), hull.end()), hull.end());
        return hull;
    }
};
C
C
#include <stdlib.h>

// Sort helper: order coordinate rows by x, then y.
static int cmpPoint(const void* a, const void* b) {

#### cons
t int* p = *(const int* const*)a;

#### cons
t int* q = *(const int* const*)b;
    if (p[0] != q[0]) return p[0] - q[0];
    return p[1] - q[1];
}

// Cross-product sign used to decide whether the last point makes a "wrong" turn.
static long cross(const int* p, const int* q, const int* r) {
    return (long)(q[1] - p[1]) * (r[0] - q[0]) - (long)(q[0] - p[0]) * (r[1] - q[1]);
}

// Returns every marker lying on the tightest enclosing boundary (perimeter, incl. edge points).
// Caller frees the returned rows, the outer array, and *returnColumnSizes.
int** boundaryPoints(int** points, int pointsSize, int* returnSize, int** returnColumnSizes) {
    if (pointsSize <= 1) {
        int** res = malloc(pointsSize * sizeof(int*));
        *returnColumnSizes = malloc(pointsSize * sizeof(int));
        for (int i = 0; i < pointsSize; i++) {
            res[i] = malloc(2 * sizeof(int));
            res[i][0] = points[i][0];
            res[i][1] = points[i][1];
            (*returnColumnSizes)[i] = 2;
        }
        *returnSize = pointsSize;
        return res;
    }

    int** pts = malloc(pointsSize * sizeof(int*));
    for (int i = 0; i < pointsSize; i++) pts[i] = points[i];
    qsort(pts, pointsSize, sizeof(int*), cmpPoint);

    int** hull = malloc((2 * pointsSize + 1) * sizeof(int*));
    int top = 0;

    // Lower pass, left to right; keep collinear points by only discarding a strict turn.
    for (int i = 0; i < pointsSize; i++) {
        while (top > 1 && cross(hull[top - 1], hull[top - 2], pts[i]) > 0) top--;
        hull[top++] = pts[i];
    }
    top--;  // drop the last point; it restarts the upper pass

    // Upper pass, right to left.
    for (int i = pointsSize - 1; i >= 0; i--) {
        while (top > 1 && cross(hull[top - 1], hull[top - 2], pts[i]) > 0) top--;
        hull[top++] = pts[i];
    }

    // The two passes can meet on shared boundary points; sort and dedup.
    qsort(hull, top, sizeof(int*), cmpPoint);
    int k = 0;
    for (int i = 0; i < top; i++)
        if (k == 0 || hull[i][0] != hull[k - 1][0] || hull[i][1] != hull[k - 1][1])
            hull[k++] = hull[i];

    int** res = malloc(k * sizeof(int*));
    *returnColumnSizes = malloc(k * sizeof(int));
    for (int i = 0; i < k; i++) {
        res[i] = malloc(2 * sizeof(int));
        res[i][0] = hull[i][0];
        res[i][1] = hull[i][1];
        (*returnColumnSizes)[i] = 2;
    }
    *returnSize = k;
    free(pts);
    free(hull);
    return res;
}

#### Follow-up
questions
What changes if you must return only the true corner vertices and exclude points lying on the interior of an edge (a standard convex hull)? Flip the pop condition to non-strict (cross >= 0).
Given the boundary, compute the total length of fence (the hull perimeter) and discuss floating-point accuracy.
The coordinates here fit in a byte, but how would you keep the cross product overflow-safe for coordinates up to 1e9?
Handle degenerate inputs explicitly: all points identical, all collinear, or fewer than three points — verify each returns the correct perimeter.

