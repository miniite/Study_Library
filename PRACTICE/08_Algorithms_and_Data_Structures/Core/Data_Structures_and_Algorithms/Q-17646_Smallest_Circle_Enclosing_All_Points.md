---
type: "practice"
difficulty: "Other"
---

# Smallest Circle Enclosing All Points

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Computational Geometry](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Computational_Geometry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 2D integer array markers, where markers[i] = [xi, yi] is the position of the i-th marker on a flat plane.

You must cover all markers with a single circular region using the smallest possible circle. A marker is considered covered if it lies inside the circle or exactly on its boundary.

Formally, find a circle with center (x, y) and radius r such that every marker satisfies distance((x, y), (xi, yi)) <= r, and r is as small as possible.

Return the center and radius as a length-3 array [x, y, r]. Any answer within 1e-5 of the true optimum (for each of the three values) is accepted.

#### Cons
traints

1 <= markers.length <= 3000
markers[i].length == 2
0 <= xi, yi <= 3000
Two markers may share the same position.

#### Example
s & test cases
TEXT
Input: markers = [[0,0],[4,0],[2,1],[2,-1]] -> Output: [2.00000, 0.00000, 2.00000]
  Why: the two farthest markers (0,0) and (4,0) are 4 apart; their midpoint (2,0) with radius 2 also covers (2,1) and (2,-1), which sit exactly distance 1 away.

Input: markers = [[1,3],[3,3],[7,3]] -> Output: [4.00000, 3.00000, 3.00000]
  Why: three collinear markers; the extreme pair (1,3) and (7,3) defines the diameter, center (4,3) radius 3, with (3,3) comfortably inside.

Input: markers = [[5,5]] -> Output: [5.00000, 5.00000, 0.00000]
  Why: a single marker needs only a degenerate circle of radius 0 centered on it.

Input: markers = [[0,0],[0,2],[2,0]] -> Output: [1.00000, 1.00000, 1.41421]
  Why: a right triangle; the hypotenuse from (0,2) to (2,0) is the diameter, so the circumcircle center (1,1) with radius sqrt(2) is minimal.

Input: markers = [[2,2],[2,2]] -> Output: [2.00000, 2.00000, 0.00000]
  Why: duplicate positions collapse to one point, so the smallest circle has radius 0.

Input: markers = [[0,0],[6,0],[6,6],[0,6]] -> Output: [3.00000, 3.00000, 4.24264]
  Why: a square's minimal enclosing circle is its circumscribed circle; center (3,3) with radius equal to half the diagonal, 3*sqrt(2).

#### Approach

This is the minimum enclosing circle (smallest bounding circle) problem, solved with Welzl's algorithm. Shuffle the points, then build the circle incrementally: keep the smallest circle enclosing all points seen so far. When a new point falls outside the current circle, that point must lie on the boundary of the new optimal circle, so restart the inner search treating it as fixed; the same logic recurses to a second and, if needed, a third boundary point. A circle is uniquely determined by two points (as a diameter) or three non-collinear points (their circumcircle). Random insertion order makes the expected cost linear. The collinear case in the three-point routine falls back to the widest defining pair.

Complexity

Time: O(n) expected, where n is the number of markers — Welzl's randomized incremental algorithm. Space: O(n) for the shuffled point copy (O(1) extra beyond that). Justification: although the three nested loops look cubic, a probabilistic argument shows a point forces re-computation of the current circle only with probability O(1/i) at level i, so the total expected work telescopes to linear.

Python
PYTHON
import random
import math

class Solution:
    def smallestEnclosingCircle(self, markers):
        def dist(ax, ay, bx, by):
            return math.hypot(ax - bx, ay - by)

        def inside(c, px, py):
            return dist(c[0], c[1], px, py) <= c[2] + 1e-7

        def from_two(ax, ay, bx, by):
            return ((ax + bx) / 2.0, (ay + by) / 2.0, dist(ax, ay, bx, by) / 2.0)

        def from_three(ax, ay, bx, by, cx, cy):
            d = 2.0 * (ax * (by - cy) + bx * (cy - ay) + cx * (ay - by))
            if abs(d) < 1e-12:                      # collinear: farthest pair
                best = from_two(ax, ay, bx, by)
                for cand in (from_two(ax, ay, cx, cy), from_two(bx, by, cx, cy)):
                    if cand[2] > best[2]:
                        best = cand
                return best
            a2 = ax * ax + ay * ay
            b2 = bx * bx + by * by
            c2 = cx * cx + cy * cy
            ux = (a2 * (by - cy) + b2 * (cy - ay) + c2 * (ay - by)) / d
            uy = (a2 * (cx - bx) + b2 * (ax - cx) + c2 * (bx - ax)) / d
            return (ux, uy, dist(ux, uy, ax, ay))

        pts = [(float(x), float(y)) for x, y in markers]
        random.seed(12345)
        random.shuffle(pts)
        c = (pts[0][0], pts[0][1], 0.0)
        for i in range(1, len(pts)):
            if inside(c, pts[i][0], pts[i][1]):
                continue
            c = (pts[i][0], pts[i][1], 0.0)
            for j in range(i):
                if inside(c, pts[j][0], pts[j][1]):
                    continue
                c = from_two(pts[i][0], pts[i][1], pts[j][0], pts[j][1])
                for k in range(j):
                    if inside(c, pts[k][0], pts[k][1]):
                        continue
                    c = from_three(pts[i][0], pts[i][1], pts[j][0], pts[j][1],
                                   pts[k][0], pts[k][1])
        return [c[0], c[1], c[2]]
JavaScript
JAVASCRIPT
function smallestEnclosingCircle(markers) {

#### cons
t dist = (ax, ay, bx, by) => Math.hypot(ax - bx, ay - by);

#### cons
t inside = (c, px, py) => dist(c[0], c[1], px, py) <= c[2] + 1e-7;

#### cons
t fromTwo = (ax, ay, bx, by) =>
        [(ax + bx) / 2, (ay + by) / 2, dist(ax, ay, bx, by) / 2];

#### cons
t fromThree = (ax, ay, bx, by, cx, cy) => {

#### cons
t d = 2 * (ax * (by - cy) + bx * (cy - ay) + cx * (ay - by));
        if (Math.abs(d) < 1e-12) {                    // collinear: farthest pair
            let best = fromTwo(ax, ay, bx, by);
            for (const cand of [fromTwo(ax, ay, cx, cy), fromTwo(bx, by, cx, cy)])
                if (cand[2] > best[2]) best = cand;
            return best;
        }

#### cons
t a2 = ax * ax + ay * ay, b2 = bx * bx + by * by, c2 = cx * cx + cy * cy;

#### cons
t ux = (a2 * (by - cy) + b2 * (cy - ay) + c2 * (ay - by)) / d;

#### cons
t uy = (a2 * (cx - bx) + b2 * (ax - cx) + c2 * (bx - ax)) / d;
        return [ux, uy, dist(ux, uy, ax, ay)];
    };

#### cons
t pts = markers.map(p => [p[0], p[1]]);
    for (let i = pts.length - 1; i > 0; i--) {        // Fisher-Yates shuffle

#### cons
t j = Math.floor(Math.random() * (i + 1));
        [pts[i], pts[j]] = [pts[j], pts[i]];
    }
    let c = [pts[0][0], pts[0][1], 0];
    for (let i = 1; i < pts.length; i++) {
        if (inside(c, pts[i][0], pts[i][1])) continue;
        c = [pts[i][0], pts[i][1], 0];
        for (let j = 0; j < i; j++) {
            if (inside(c, pts[j][0], pts[j][1])) continue;
            c = fromTwo(pts[i][0], pts[i][1], pts[j][0], pts[j][1]);
            for (let k = 0; k < j; k++) {
                if (inside(c, pts[k][0], pts[k][1])) continue;
                c = fromThree(pts[i][0], pts[i][1], pts[j][0], pts[j][1],
                              pts[k][0], pts[k][1]);
            }
        }
    }
    return [c[0], c[1], c[2]];
}
Java
JAVA
import java.util.*;

class Solution {
    static class Circle {
        double x, y, r;
        Circle(double x, double y, double r) { this.x = x; this.y = y; this.r = r; }
    }

    static double dist(double ax, double ay, double bx, double by) {
        double dx = ax - bx, dy = ay - by;
        return Math.sqrt(dx * dx + dy * dy);
    }
    static boolean inside(Circle c, double px, double py) {
        return dist(c.x, c.y, px, py) <= c.r + 1e-7;
    }
    static Circle fromTwo(double ax, double ay, double bx, double by) {
        return new Circle((ax + bx) / 2.0, (ay + by) / 2.0, dist(ax, ay, bx, by) / 2.0);
    }
    static Circle fromThree(double ax, double ay, double bx, double by, double cx, double cy) {
        double d = 2.0 * (ax * (by - cy) + bx * (cy - ay) + cx * (ay - by));
        if (Math.abs(d) < 1e-12) {
            Circle best = fromTwo(ax, ay, bx, by);
            Circle c2 = fromTwo(ax, ay, cx, cy);
            Circle c3 = fromTwo(bx, by, cx, cy);
            if (c2.r > best.r) best = c2;
            if (c3.r > best.r) best = c3;
            return best;
        }
        double a2 = ax * ax + ay * ay, b2 = bx * bx + by * by, cc2 = cx * cx + cy * cy;
        double ux = (a2 * (by - cy) + b2 * (cy - ay) + cc2 * (ay - by)) / d;
        double uy = (a2 * (cx - bx) + b2 * (ax - cx) + cc2 * (bx - ax)) / d;
        return new Circle(ux, uy, dist(ux, uy, ax, ay));
    }

    public double[] smallestEnclosingCircle(int[][] markers) {
        int n = markers.length;
        double[][] pts = new double[n][2];
        for (int i = 0; i < n; i++) { pts[i][0] = markers[i][0]; pts[i][1] = markers[i][1]; }
        Random rng = new Random(12345);
        for (int i = n - 1; i > 0; i--) {
            int j = rng.nextInt(i + 1);
            double[] t = pts[i]; pts[i] = pts[j]; pts[j] = t;
        }
        Circle c = new Circle(pts[0][0], pts[0][1], 0.0);
        for (int i = 1; i < n; i++) {
            if (inside(c, pts[i][0], pts[i][1])) continue;
            c = new Circle(pts[i][0], pts[i][1], 0.0);
            for (int j = 0; j < i; j++) {
                if (inside(c, pts[j][0], pts[j][1])) continue;
                c = fromTwo(pts[i][0], pts[i][1], pts[j][0], pts[j][1]);
                for (int k = 0; k < j; k++) {
                    if (inside(c, pts[k][0], pts[k][1])) continue;
                    c = fromThree(pts[i][0], pts[i][1], pts[j][0], pts[j][1],
                                  pts[k][0], pts[k][1]);
                }
            }
        }
        return new double[]{ c.x, c.y, c.r };
    }
}
C++
CPP
#include <vector>
#include <random>
#include <cmath>
using namespace std;

class Solution {
    struct Circle { double x, y, r; };

    static double dist(double ax, double ay, double bx, double by) {
        double dx = ax - bx, dy = ay - by;
        return sqrt(dx * dx + dy * dy);
    }
    static bool inside(const Circle& c, double px, double py) {
        return dist(c.x, c.y, px, py) <= c.r + 1e-7;
    }
    static Circle fromTwo(double ax, double ay, double bx, double by) {
        return { (ax + bx) / 2.0, (ay + by) / 2.0, dist(ax, ay, bx, by) / 2.0 };
    }
    static Circle fromThree(double ax, double ay, double bx, double by, double cx, double cy) {
        double d = 2.0 * (ax * (by - cy) + bx * (cy - ay) + cx * (ay - by));
        if (fabs(d) < 1e-12) {
            Circle best = fromTwo(ax, ay, bx, by);
            Circle c2 = fromTwo(ax, ay, cx, cy);
            Circle c3 = fromTwo(bx, by, cx, cy);
            if (c2.r > best.r) best = c2;
            if (c3.r > best.r) best = c3;
            return best;
        }
        double a2 = ax * ax + ay * ay, b2 = bx * bx + by * by, c2 = cx * cx + cy * cy;
        double ux = (a2 * (by - cy) + b2 * (cy - ay) + c2 * (ay - by)) / d;
        double uy = (a2 * (cx - bx) + b2 * (ax - cx) + c2 * (bx - ax)) / d;
        return { ux, uy, dist(ux, uy, ax, ay) };
    }

public:
    vector<double> smallestEnclosingCircle(vector<vector<int>>& markers) {
        int n = (int)markers.size();
        vector<pair<double,double>> pts;
        pts.reserve(n);
        for (auto& p : markers) pts.push_back({ (double)p[0], (double)p[1] });
        mt19937 rng(12345);
        shuffle(pts.begin(), pts.end(), rng);
        Circle c = { pts[0].first, pts[0].second, 0.0 };
        for (int i = 1; i < n; i++) {
            if (inside(c, pts[i].first, pts[i].second)) continue;
            c = { pts[i].first, pts[i].second, 0.0 };
            for (int j = 0; j < i; j++) {
                if (inside(c, pts[j].first, pts[j].second)) continue;
                c = fromTwo(pts[i].first, pts[i].second, pts[j].first, pts[j].second);
                for (int k = 0; k < j; k++) {
                    if (inside(c, pts[k].first, pts[k].second)) continue;
                    c = fromThree(pts[i].first, pts[i].second,
                                  pts[j].first, pts[j].second,
                                  pts[k].first, pts[k].second);
                }
            }
        }
        return { c.x, c.y, c.r };
    }
};
C
C
#include <stdlib.h>
#include <math.h>

typedef struct { double x, y, r; } Circle;

static double dist(double ax, double ay, double bx, double by) {
    double dx = ax - bx, dy = ay - by;
    return sqrt(dx * dx + dy * dy);
}
static int inside(Circle c, double px, double py) {
    return dist(c.x, c.y, px, py) <= c.r + 1e-7;
}
static Circle fromTwo(double ax, double ay, double bx, double by) {
    Circle c;
    c.x = (ax + bx) / 2.0;
    c.y = (ay + by) / 2.0;
    c.r = dist(ax, ay, bx, by) / 2.0;
    return c;
}
static Circle fromThree(double ax, double ay, double bx, double by, double cx, double cy) {
    double d = 2.0 * (ax * (by - cy) + bx * (cy - ay) + cx * (ay - by));
    if (fabs(d) < 1e-12) {                 /* collinear: use farthest pair */
        Circle best = fromTwo(ax, ay, bx, by);
        Circle c2 = fromTwo(ax, ay, cx, cy);
        Circle c3 = fromTwo(bx, by, cx, cy);
        if (c2.r > best.r) best = c2;
        if (c3.r > best.r) best = c3;
        return best;
    }
    double a2 = ax * ax + ay * ay, b2 = bx * bx + by * by, c2 = cx * cx + cy * cy;
    Circle c;
    c.x = (a2 * (by - cy) + b2 * (cy - ay) + c2 * (ay - by)) / d;
    c.y = (a2 * (cx - bx) + b2 * (ax - cx) + c2 * (bx - ax)) / d;
    c.r = dist(c.x, c.y, ax, ay);
    return c;
}

double* smallestEnclosingCircle(int** markers, int markersSize, int* markersColSize,
                                int* returnSize) {
    (void)markersColSize;
    double* xs = (double*)malloc(sizeof(double) * markersSize);
    double* ys = (double*)malloc(sizeof(double) * markersSize);
    for (int i = 0; i < markersSize; i++) { xs[i] = markers[i][0]; ys[i] = markers[i][1]; }
    srand(12345);                          /* randomized incremental order */
    for (int i = markersSize - 1; i > 0; i--) {
        int j = rand() % (i + 1);
        double t;
        t = xs[i]; xs[i] = xs[j]; xs[j] = t;
        t = ys[i]; ys[i] = ys[j]; ys[j] = t;
    }
    Circle c; c.x = xs[0]; c.y = ys[0]; c.r = 0.0;
    for (int i = 1; i < markersSize; i++) {
        if (inside(c, xs[i], ys[i])) continue;
        c.x = xs[i]; c.y = ys[i]; c.r = 0.0;
        for (int j = 0; j < i; j++) {
            if (inside(c, xs[j], ys[j])) continue;
            c = fromTwo(xs[i], ys[i], xs[j], ys[j]);
            for (int k = 0; k < j; k++) {
                if (inside(c, xs[k], ys[k])) continue;
                c = fromThree(xs[i], ys[i], xs[j], ys[j], xs[k], ys[k]);
            }
        }
    }
    free(xs); free(ys);
    double* res = (double*)malloc(sizeof(double) * 3);
    res[0] = c.x; res[1] = c.y; res[2] = c.r;
    *returnSize = 3;
    return res;
}

#### Follow-up
questions
Why is Welzl's algorithm expected O(n) despite the nested loops — can you sketch the probability argument?
How would you extend this to three dimensions (smallest enclosing sphere)?
If the input can be adversarial, how do random shuffling and floating-point tolerance protect correctness?
Could you instead solve it with a convex-hull step followed by rotating the enclosing circle, and how does that compare in complexity?

