---
type: "practice"
difficulty: "Other"
---

# Minimum Straight Lines to Draw a Polyline

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Math](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Math.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list of 2D points, where points[i] = [x_i, y_i]. All x-coordinates are distinct.

Sort the points by their x-coordinate and connect every pair of adjacent points (in that sorted order) with a straight segment, forming one connected polyline. Whenever several consecutive segments happen to lie on the same straight line, they can be drawn together as a single line instead of as separate pieces.

Return the minimum number of straight lines needed to draw the entire polyline. If there are fewer than two points there is nothing to connect, so return 0.

#### Cons
traints:

1 <= points.length <= 10^5
points[i].length == 2
1 <= x_i, y_i <= 10^9
All x_i are distinct.

#### Example
s & test cases
TEXT
Input: points = [[2,5],[4,3],[6,1],[3,4]] -> Output: 1
Why: sorted by x the points are (2,5),(3,4),(4,3),(6,1); every consecutive segment has slope -1, so a single straight line covers them all.

Input: points = [[1,1],[2,2],[3,4],[4,5]] -> Output: 3
Why: sorted order (1,1),(2,2),(3,4),(4,5) bends at (2,2) and again at (3,4); three distinct segment directions mean three separate lines.

Input: points = [[5,9]] -> Output: 0
Why: a single point cannot form any segment, so no line is needed — the minimum-size edge case.

Input: points = [[10,1],[1,10]] -> Output: 1
Why: any two points define exactly one straight line; sorting first does not change that.

Input: points = [[1,7],[2,6],[3,5],[4,4],[5,4],[6,3],[7,2],[8,1]] -> Output: 3
Why: slope -1 for the first four points, then a flat segment from (4,4) to (5,4), then slope -1 again — three runs of collinear segments, so three lines.

#### Approach

Sort the points by x-coordinate so the polyline is well defined. A new straight line is required exactly whenever the direction of the incoming segment differs from that of the outgoing one, so walk over consecutive triples and count the direction changes; the answer is one line for the first segment plus one for every change (and 0 when there are fewer than two points). To test whether three consecutive points are collinear without floating-point error or division-by-zero on vertical steps, compare slopes by cross-multiplication: segments (a→b) and (b→c) share a direction iff (bx-ax)*(cy-by) == (cx-bx)*(by-ay). Because coordinates reach 1e9, those products reach ~1e18, so the arithmetic must use 64-bit integers (and BigInt in JavaScript). The pattern is a slope-run / collinearity sweep using an integer cross product.

Complexity

Time: O(n log n) — sorting the points by x dominates; the single pass counting direction changes is O(n). Space: O(1) auxiliary beyond the sort (the sort may use O(log n) to O(n) internally depending on the implementation).

Python
PYTHON
def minimum_lines(points):
    if len(points) <= 1:
        return 0
    points.sort()
    lines = 1
    for i in range(2, len(points)):
        dx1 = points[i - 1][0] - points[i - 2][0]
        dy1 = points[i - 1][1] - points[i - 2][1]
        dx2 = points[i][0] - points[i - 1][0]
        dy2 = points[i][1] - points[i - 1][1]
        if dx1 * dy2 != dx2 * dy1:
            lines += 1
    return lines
JavaScript
JAVASCRIPT
function minimumLines(points) {
    if (points.length <= 1) return 0;
    points.sort((a, b) => a[0] - b[0]);

    let lines = 1;
    for (let i = 2; i < points.length; i++) {
        // Use BigInt: coordinate differences can multiply to ~1e18, beyond
        // Number's safe-integer range, so plain Number could misjudge collinearity.

#### cons
t dx1 = BigInt(points[i - 1][0] - points[i - 2][0]);

#### cons
t dy1 = BigInt(points[i - 1][1] - points[i - 2][1]);

#### cons
t dx2 = BigInt(points[i][0] - points[i - 1][0]);

#### cons
t dy2 = BigInt(points[i][1] - points[i - 1][1]);
        if (dx1 * dy2 !== dx2 * dy1) lines++;
    }
    return lines;
}
Java
JAVA
import java.util.Arrays;

class Solution {
    public int minimumLines(int[][] points) {
        if (points.length <= 1) return 0;
        Arrays.sort(points, (a, b) -> Integer.compare(a[0], b[0]));

        int lines = 1;
        for (int i = 2; i < points.length; i++) {
            long dx1 = (long) points[i - 1][0] - points[i - 2][0];
            long dy1 = (long) points[i - 1][1] - points[i - 2][1];
            long dx2 = (long) points[i][0]     - points[i - 1][0];
            long dy2 = (long) points[i][1]     - points[i - 1][1];
            if (dx1 * dy2 != dx2 * dy1) lines++;
        }
        return lines;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int minimumLines(vector<vector<int>>& points) {
        if (points.size() <= 1) return 0;
        sort(points.begin(), points.end());

        int lines = 1;
        for (size_t i = 2; i < points.size(); i++) {
            long long dx1 = (long long)points[i - 1][0] - points[i - 2][0];
            long long dy1 = (long long)points[i - 1][1] - points[i - 2][1];
            long long dx2 = (long long)points[i][0]     - points[i - 1][0];
            long long dy2 = (long long)points[i][1]     - points[i - 1][1];
            if (dx1 * dy2 != dx2 * dy1) lines++;
        }
        return lines;
    }
};
C
C
#include <stdlib.h>

static int cmp_pts(const void *a, const void *b) {
    int *pa = *(int **)a;
    int *pb = *(int **)b;
    return (pa[0] > pb[0]) - (pa[0] < pb[0]);
}

int minimumLines(int **points, int pointsSize, int *pointsColSize) {
    if (pointsSize <= 1) return 0;
    qsort(points, pointsSize, sizeof(int *), cmp_pts);

    int lines = 1;
    for (int i = 2; i < pointsSize; i++) {
        long long dx1 = (long long)points[i - 1][0] - points[i - 2][0];
        long long dy1 = (long long)points[i - 1][1] - points[i - 2][1];
        long long dx2 = (long long)points[i][0]     - points[i - 1][0];
        long long dy2 = (long long)points[i][1]     - points[i - 1][1];
        if (dx1 * dy2 != dx2 * dy1) lines++;
    }
    return lines;
}

#### Follow-up
questions
Why is comparing slopes with cross-multiplication preferable to computing dy/dx directly? (avoids floating-point error and division by zero on vertical steps)
Given the stated bounds, what integer width does the cross product require, and what goes wrong with a 32-bit or plain double?
How does the problem change if x-coordinates are allowed to repeat, so vertical segments can occur?
Could you process the points in a stream that is already sorted by x and report the count online with O(1) memory?

