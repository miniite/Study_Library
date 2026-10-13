---
type: "practice"
difficulty: "Other"
---

# Longest Ready Prefix Tracker

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A digitization workshop scans the pages of a manuscript one at a time. The manuscript has n pages, numbered from 1 to n. Pages are scanned in an arbitrary order (not necessarily 1, 2, 3, …), and every page number is scanned at most once.

At any moment a supervisor wants to know how much of the manuscript is continuously ready from the beginning. We say the workshop has a ready prefix of length i when every page in the range 1 through i (inclusive) has already been scanned. The longest ready prefix is the largest such i; if page 1 has not been scanned yet, the longest ready prefix has length 0.

Design a data structure that supports these operations efficiently:

PrefixTracker(int n) — initialize the tracker for a manuscript of n pages, with nothing scanned yet.
void receive(int page) — record that page has just been scanned.
int longestReady() — return the length of the current longest ready prefix.

Input / Output

The constructor receives n.
Each receive(page) returns nothing; it updates internal state.
Each longestReady() returns an integer — the length of the longest contiguous prefix 1..i that is fully scanned.

#### Cons
traints

1 <= n <= 10^5
1 <= page <= n
All values passed to receive are distinct.
At most 2 * 10^5 calls in total are made across receive and longestReady.
At least one call to longestReady is made.

#### Example
s & test cases
TEXT
Input: PrefixTracker(4); receive(3); longestReady() -> Output: 0
Why: page 1 is missing, so no prefix starting at 1 is complete.

Input: (continuing) receive(1); longestReady() -> Output: 1
Why: pages {1,3} scanned; only [1] is contiguous from the start, so length 1.

Input: (continuing) receive(2); longestReady() -> Output: 3
Why: pages {1,2,3} now form a contiguous block, extending the prefix to 3.

Input: PrefixTracker(1); receive(1); longestReady() -> Output: 1
Why: single-page manuscript fully scanned; minimal boundary case.

Input: PrefixTracker(5); longestReady() -> Output: 0
Why: nothing scanned yet, so the ready prefix is empty (length 0).

Input: PrefixTracker(3); receive(2); receive(3); longestReady(); receive(1); longestReady() -> Output: 0 then 3
Why: {2,3} leaves page 1 missing (0); once page 1 arrives the whole run 1..3 becomes ready (3).

#### Approach

Keep a boolean array marking which pages have been scanned, plus a monotonic pointer next equal to the smallest page number that is NOT yet covered by the ready prefix. The answer to longestReady is always next - 1. A newly scanned page can only extend the prefix if it is exactly next; in that case walk next forward over the already-scanned run. Because next never decreases, the amortized cost per update is constant. This is the "greedy contiguous-frontier pointer" pattern — a lightweight alternative to a Fenwick/segment tree or a union-find, which are only needed if pages could be un-scanned or if arbitrary range queries were required.

Complexity

Time: constructor O(n); longestReady O(1); receive is amortized O(1) — the next pointer only ever moves forward and advances at most n times over the whole run, so the total work in all while loops is O(n). Space: O(n) for the boolean array.

Python
PYTHON
class PrefixTracker:
    def __init__(self, n: int):
        self.n = n
        self.scanned = [False] * (n + 2)
        self.next = 1  # smallest page not yet in the ready prefix

    def receive(self, page: int) -> None:
        self.scanned[page] = True
        if page == self.next:
            while self.next <= self.n and self.scanned[self.next]:
                self.next += 1

    def longest_ready(self) -> int:
        return self.next - 1
JavaScript
JAVASCRIPT
class PrefixTracker {

#### cons
tructor(n) {
        this.n = n;
        this.scanned = new Array(n + 2).fill(false);
        this.next = 1; // smallest page not yet in the ready prefix
    }

    receive(page) {
        this.scanned[page] = true;
        if (page === this.next) {
            while (this.next <= this.n && this.scanned[this.next]) this.next++;
        }
    }

    longestReady() {
        return this.next - 1;
    }
}
Java
JAVA
class PrefixTracker {
    private final boolean[] scanned;
    private int next;   // smallest page not yet in the ready prefix
    private final int n;

    public PrefixTracker(int n) {
        this.n = n;
        this.scanned = new boolean[n + 2];
        this.next = 1;
    }

    public void receive(int page) {
        scanned[page] = true;
        if (page == next) {
            while (next <= n && scanned[next]) next++;
        }
    }

    public int longestReady() {
        return next - 1;
    }
}
C++
CPP
#include <vector>
using namespace std;

class PrefixTracker {
    vector<bool> scanned;
    int next;   // smallest page not yet in the ready prefix
    int n;
public:
    PrefixTracker(int n) : scanned(n + 2, false), next(1), n(n) {}

    void receive(int page) {
        scanned[page] = true;
        if (page == next) {
            while (next <= n && scanned[next]) next++;
        }
    }

    int longestReady() {
        return next - 1;
    }
};
C
C
#include <stdlib.h>
#include <stdbool.h>

typedef struct {
    bool *scanned;
    int   next;   /* smallest page number not yet part of the ready prefix */
    int   n;
} PrefixTracker;

PrefixTracker* prefixTrackerCreate(int n) {
    PrefixTracker *t = (PrefixTracker*)malloc(sizeof(PrefixTracker));
    t->scanned = (bool*)calloc((size_t)n + 2, sizeof(bool));
    t->next = 1;
    t->n = n;
    return t;
}

void prefixTrackerReceive(PrefixTracker *t, int page) {
    t->scanned[page] = true;
    if (page == t->next) {
        while (t->next <= t->n && t->scanned[t->next]) t->next++;
    }
}

int prefixTrackerLongestReady(PrefixTracker *t) {
    return t->next - 1;
}

void prefixTrackerFree(PrefixTracker *t) {
    free(t->scanned);
    free(t);
}

#### Follow-up
questions
What changes if pages can be un-scanned (removed) as well? (The forward-only pointer breaks; use an ordered set or a Fenwick tree over prefix sums / a union-find with path relinking.)
Support a query for the longest ready prefix ending at any position, not just from page 1.
Handle duplicate receive calls gracefully and bound total memory when n is huge but few pages arrive (switch the boolean array to a hash set).
Extend longestReady to also report the count of fully contiguous gaps remaining.

