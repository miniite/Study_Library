---
type: "practice"
difficulty: "Other"
---

# Design a Row-Based Seat Reservation System

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Segment Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Segment_Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A performance hall has rows rows, numbered from 0 to rows - 1. Every row has exactly cap seats, numbered from 0 to cap - 1. Design a reservation system that serves parties of guests under two policies.

Each row always fills from its lowest-numbered empty seat, so at any moment the occupied seats of a row form a contiguous block [0, t - 1] and the empty seats form the suffix [t, cap - 1].

Implement a class SeatReservation:

SeatReservation(int rows, int cap) — initializes the hall with rows rows and cap seats per row (every seat empty).

int[] together(int k, int limit) — seats a party of k guests in one row, in consecutive seats. The party will only accept a row whose index is <= limit. Among all rows 0..limit that currently have at least k consecutive empty seats, choose the one with the smallest row index, and within it the smallest starting seat. Book those k seats and return [row, startSeat], where startSeat is the seat number of the first seat given to the party. If no such row exists, return an empty array [].

boolean split(int k, int limit) — seats a party of k guests using any empty seats in rows 0..limit; the guests need not sit together. If the total number of empty seats in rows 0..limit is at least k, book exactly k of them, always preferring the smallest row indices first and the smallest seat numbers within each chosen row, then return true. If there are fewer than k empty seats in rows 0..limit, book nothing and return false.

#### Cons
traints
1 <= rows <= 5 * 10^4
1 <= cap, k <= 10^9
0 <= limit <= rows - 1
At most 5 * 10^4 calls in total are made to together and split.
Seat and row counts and running sums can exceed 32-bit range; use 64-bit arithmetic where sums are involved.

#### Example
s & test cases
TEXT
Input:
ops = ["SeatReservation","together","together","split","split"]
args = [[3,4],[3,0],[2,0],[6,2],[4,2]]
Output: [null,[0,0],[],true,false]
Why: Hall of 3 rows x 4 seats.
- together(3,0): row 0 has 4 empty seats; book seats 0..2 -> return [0,0]. Row 0 now has 1 empty seat.
- together(2,0): only row 0 is allowed (limit 0) and it has just 1 empty seat < 2 -> return [].
- split(6,2): empty seats in rows 0..2 = 1 + 4 + 4 = 9 >= 6. Fill row 0's last seat (1), all of row 1 (4), one seat of row 2 (1) = 6 -> return true.
- split(4,2): remaining empty in rows 0..2 = 0 + 0 + 3 = 3 < 4 -> book nothing, return false.

Input:
ops = ["SeatReservation","together","split"]
args = [[1,5],[5,0],[1,0]]
Output: [null,[0,0],false]
Why: Single row of 5 seats. together(5,0) fills the whole row and returns [0,0]. split(1,0) finds 0 empty seats -> false. Edge case: one row, exact fit.

Input:
ops = ["SeatReservation","split","together","together"]
args = [[2,3],[3,1],[2,1],[2,0]]
Output: [null,true,[1,1],[]]
Why: 2 rows x 3 seats. split(3,1): 6 empty >= 3, fills row 0 entirely (3 seats). together(2,1): row 0 full, row 1 has 3 empty consecutive -> smallest row is 1, start seat 0... wait row 1 empty from seat 0, so returns [1,0]? Trace precisely: after split(3,1) row 0 full (seats 0..2), row 1 empty. together(2,1) -> row 0 has 0 empty, row 1 has 3 empty starting at seat 0 -> [1,0]. together(2,0) -> only row 0 allowed, it is full -> []. Corrected Output: [null,true,[1,0],[]].

#### Approach

Maintain a segment tree indexed by row where each leaf stores the number of empty seats in that row (initially cap). Each internal node keeps two aggregates over its range: the maximum leaf value (drives together) and the sum of leaf values (drives split).

together(k, limit): descend the tree to find the leftmost leaf in rows [0, limit] whose stored max is >= k, pruning any subtree whose max is below k. Because each row's occupied seats form a left prefix, the first free seat in the chosen row is cap - remaining; book k seats there and update the leaf.

split(k, limit): first check the range sum over [0, limit]; if it is < k, book nothing and return false. Otherwise greedily fill from a pointer minr, the smallest row index that could still hold empty seats. Rows below minr were completely filled by earlier splits and can never regain seats (there is no un-booking) and together only ever touches rows at or above minr, so minr is a valid, monotonically increasing lower bound. Take whole rows until the last, partial row, updating each touched leaf. Since each fully consumed row advances minr exactly once, the pointer amortizes to O(rows) total work. The core pattern is a segment tree carrying two commutative aggregates (max + sum) combined with an amortized forward pointer.

Complexity

#### Cons
truction: O(rows) to build the tree. together: O(log rows) — one pruned descent that finds the leftmost qualifying row and one point update. split: O(log rows) for the range-sum feasibility check plus the greedy fill; the monotonic pointer minr only advances forward, so across ALL split calls the fully-consumed rows total at most O(rows), giving O((rows + Q) log rows) overall for Q operations. Space: O(rows) for the tree and per-row remaining array. 64-bit sums are required because total empty seats can reach rows * cap ~ 5 * 10^13.

Python
PYTHON
class SeatReservation:
    def __init__(self, rows, cap):
        self.rows = rows
        self.cap = cap
        self.rem = [cap] * rows
        self.mx = [0] * (4 * rows)
        self.sum = [0] * (4 * rows)
        self.minr = 0
        self._build(1, 0, rows - 1)

    def _build(self, node, lo, hi):
        if lo == hi:
            self.mx[node] = self.sum[node] = self.cap
            return
        mid = (lo + hi) // 2
        self._build(2 * node, lo, mid)
        self._build(2 * node + 1, mid + 1, hi)
        self.mx[node] = max(self.mx[2 * node], self.mx[2 * node + 1])
        self.sum[node] = self.sum[2 * node] + self.sum[2 * node + 1]

    def _update(self, node, lo, hi, pos, val):
        if lo == hi:
            self.mx[node] = self.sum[node] = val
            return
        mid = (lo + hi) // 2
        if pos <= mid:
            self._update(2 * node, lo, mid, pos, val)
        else:
            self._update(2 * node + 1, mid + 1, hi, pos, val)
        self.mx[node] = max(self.mx[2 * node], self.mx[2 * node + 1])
        self.sum[node] = self.sum[2 * node] + self.sum[2 * node + 1]

    def _first_ge(self, node, lo, hi, limit, k):
        if lo > limit or self.mx[node] < k:
            return -1
        if lo == hi:
            return lo
        mid = (lo + hi) // 2
        left = self._first_ge(2 * node, lo, mid, limit, k)
        if left != -1:
            return left
        return self._first_ge(2 * node + 1, mid + 1, hi, limit, k)

    def _range_sum(self, node, lo, hi, l, r):
        if r < lo or hi < l:
            return 0
        if l <= lo and hi <= r:
            return self.sum[node]
        mid = (lo + hi) // 2
        return (self._range_sum(2 * node, lo, mid, l, r)
                + self._range_sum(2 * node + 1, mid + 1, hi, l, r))

    def together(self, k, limit):
        r = self._first_ge(1, 0, self.rows - 1, limit, k)
        if r == -1:
            return []
        start = self.cap - self.rem[r]
        self.rem[r] -= k
        self._update(1, 0, self.rows - 1, r, self.rem[r])
        return [r, start]

    def split(self, k, limit):
        need = k
        if self._range_sum(1, 0, self.rows - 1, 0, limit) < need:
            return False
        i = self.minr
        while need > 0:
            rem = self.rem[i]
            if rem <= 0:
                i += 1
                self.minr = i
                continue
            if rem <= need:
                need -= rem
                self.rem[i] = 0
                self._update(1, 0, self.rows - 1, i, 0)
                i += 1
                self.minr = i
            else:
                self.rem[i] = rem - need
                self._update(1, 0, self.rows - 1, i, self.rem[i])
                need = 0
        return True
JavaScript
JAVASCRIPT
class SeatReservation {

#### cons
tructor(rows, cap) {
        this.rows = rows;
        this.cap = cap;
        this.rem = new Array(rows).fill(cap);
        this.mx = new Array(4 * rows).fill(0);
        this.sum = new Array(4 * rows).fill(0);
        this.minr = 0;
        this._build(1, 0, rows - 1);
    }
    _build(node, lo, hi) {
        if (lo === hi) { this.mx[node] = this.sum[node] = this.cap; return; }

#### cons
t mid = (lo + hi) >> 1;
        this._build(2 * node, lo, mid);
        this._build(2 * node + 1, mid + 1, hi);
        this.mx[node] = Math.max(this.mx[2 * node], this.mx[2 * node + 1]);
        this.sum[node] = this.sum[2 * node] + this.sum[2 * node + 1];
    }
    _update(node, lo, hi, pos, val) {
        if (lo === hi) { this.mx[node] = this.sum[node] = val; return; }

#### cons
t mid = (lo + hi) >> 1;
        if (pos <= mid) this._update(2 * node, lo, mid, pos, val);
        else this._update(2 * node + 1, mid + 1, hi, pos, val);
        this.mx[node] = Math.max(this.mx[2 * node], this.mx[2 * node + 1]);
        this.sum[node] = this.sum[2 * node] + this.sum[2 * node + 1];
    }
    _firstGE(node, lo, hi, limit, k) {
        if (lo > limit || this.mx[node] < k) return -1;
        if (lo === hi) return lo;

#### cons
t mid = (lo + hi) >> 1;

#### cons
t left = this._firstGE(2 * node, lo, mid, limit, k);
        if (left !== -1) return left;
        return this._firstGE(2 * node + 1, mid + 1, hi, limit, k);
    }
    _rangeSum(node, lo, hi, l, r) {
        if (r < lo || hi < l) return 0;
        if (l <= lo && hi <= r) return this.sum[node];

#### cons
t mid = (lo + hi) >> 1;
        return this._rangeSum(2 * node, lo, mid, l, r)
             + this._rangeSum(2 * node + 1, mid + 1, hi, l, r);
    }
    together(k, limit) {

#### cons
t r = this._firstGE(1, 0, this.rows - 1, limit, k);
        if (r === -1) return [];

#### cons
t start = this.cap - this.rem[r];
        this.rem[r] -= k;
        this._update(1, 0, this.rows - 1, r, this.rem[r]);
        return [r, start];
    }
    split(k, limit) {
        let need = k;
        if (this._rangeSum(1, 0, this.rows - 1, 0, limit) < need) return false;
        let i = this.minr;
        while (need > 0) {

#### cons
t rem = this.rem[i];
            if (rem <= 0) { this.minr = ++i; continue; }
            if (rem <= need) {
                need -= rem;
                this.rem[i] = 0;
                this._update(1, 0, this.rows - 1, i, 0);
                this.minr = ++i;
            } else {
                this.rem[i] = rem - need;
                this._update(1, 0, this.rows - 1, i, this.rem[i]);
                need = 0;
            }
        }
        return true;
    }
}
Java
JAVA
class SeatReservation {
    private final int rows;
    private final long cap;
    private final long[] rem, mx, sum;
    private int minr = 0;

    public SeatReservation(int rows, int cap) {
        this.rows = rows;
        this.cap = cap;
        rem = new long[rows];
        java.util.Arrays.fill(rem, cap);
        mx = new long[4 * rows];
        sum = new long[4 * rows];
        build(1, 0, rows - 1);
    }
    private void build(int node, int lo, int hi) {
        if (lo == hi) { mx[node] = sum[node] = cap; return; }
        int mid = (lo + hi) / 2;
        build(2*node, lo, mid);
        build(2*node+1, mid+1, hi);
        mx[node] = Math.max(mx[2*node], mx[2*node+1]);
        sum[node] = sum[2*node] + sum[2*node+1];
    }
    private void update(int node, int lo, int hi, int pos, long val) {
        if (lo == hi) { mx[node] = sum[node] = val; return; }
        int mid = (lo + hi) / 2;
        if (pos <= mid) update(2*node, lo, mid, pos, val);
        else update(2*node+1, mid+1, hi, pos, val);
        mx[node] = Math.max(mx[2*node], mx[2*node+1]);
        sum[node] = sum[2*node] + sum[2*node+1];
    }
    private int firstGE(int node, int lo, int hi, int limit, long k) {
        if (lo > limit || mx[node] < k) return -1;
        if (lo == hi) return lo;
        int mid = (lo + hi) / 2;
        int left = firstGE(2*node, lo, mid, limit, k);
        if (left != -1) return left;
        return firstGE(2*node+1, mid+1, hi, limit, k);
    }
    private long rangeSum(int node, int lo, int hi, int l, int r) {
        if (r < lo || hi < l) return 0;
        if (l <= lo && hi <= r) return sum[node];
        int mid = (lo + hi) / 2;
        return rangeSum(2*node, lo, mid, l, r) + rangeSum(2*node+1, mid+1, hi, l, r);
    }
    public int[] together(int k, int limit) {
        int r = firstGE(1, 0, rows - 1, limit, k);
        if (r == -1) return new int[0];
        long start = cap - rem[r];
        rem[r] -= k;
        update(1, 0, rows - 1, r, rem[r]);
        return new int[]{r, (int) start};
    }
    public boolean split(int k, int limit) {
        long need = k;
        if (rangeSum(1, 0, rows - 1, 0, limit) < need) return false;
        int i = minr;
        while (need > 0) {
            long r = rem[i];
            if (r <= 0) { minr = ++i; continue; }
            if (r <= need) {
                need -= r; rem[i] = 0;
                update(1, 0, rows - 1, i, 0);
                minr = ++i;
            } else {
                rem[i] = r - need;
                update(1, 0, rows - 1, i, rem[i]);
                need = 0;
            }
        }
        return true;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class SeatReservation {
    int rows;
    long long cap;
    vector<long long> rem, mx, sum;
    int minr = 0;

    void build(int node, int lo, int hi) {
        if (lo == hi) { mx[node] = sum[node] = cap; return; }
        int mid = (lo + hi) / 2;
        build(2*node, lo, mid);
        build(2*node+1, mid+1, hi);
        mx[node] = max(mx[2*node], mx[2*node+1]);
        sum[node] = sum[2*node] + sum[2*node+1];
    }
    void update(int node, int lo, int hi, int pos, long long val) {
        if (lo == hi) { mx[node] = sum[node] = val; return; }
        int mid = (lo + hi) / 2;
        if (pos <= mid) update(2*node, lo, mid, pos, val);
        else update(2*node+1, mid+1, hi, pos, val);
        mx[node] = max(mx[2*node], mx[2*node+1]);
        sum[node] = sum[2*node] + sum[2*node+1];
    }
    int firstGE(int node, int lo, int hi, int limit, long long k) {
        if (lo > limit || mx[node] < k) return -1;
        if (lo == hi) return lo;
        int mid = (lo + hi) / 2;
        int left = firstGE(2*node, lo, mid, limit, k);
        if (left != -1) return left;
        return firstGE(2*node+1, mid+1, hi, limit, k);
    }
    long long rangeSum(int node, int lo, int hi, int l, int r) {
        if (r < lo || hi < l) return 0;
        if (l <= lo && hi <= r) return sum[node];
        int mid = (lo + hi) / 2;
        return rangeSum(2*node, lo, mid, l, r) + rangeSum(2*node+1, mid+1, hi, l, r);
    }
public:
    SeatReservation(int rows, int cap)
        : rows(rows), cap(cap), rem(rows, cap), mx(4*rows), sum(4*rows) {
        build(1, 0, rows - 1);
    }
    vector<int> together(int k, int limit) {
        int r = firstGE(1, 0, rows - 1, limit, k);
        if (r == -1) return {};
        long long start = cap - rem[r];
        rem[r] -= k;
        update(1, 0, rows - 1, r, rem[r]);
        return {r, (int)start};
    }
    bool split(int k, int limit) {
        long long need = k;
        if (rangeSum(1, 0, rows - 1, 0, limit) < need) return false;
        int i = minr;
        while (need > 0) {
            long long r = rem[i];
            if (r <= 0) { minr = ++i; continue; }
            if (r <= need) {
                need -= r; rem[i] = 0;
                update(1, 0, rows - 1, i, 0);
                minr = ++i;
            } else {
                rem[i] = r - need;
                update(1, 0, rows - 1, i, rem[i]);
                need = 0;
            }
        }
        return true;
    }
};
C
C
#include <stdlib.h>

typedef struct {
    int rows;
    long long cap;
    long long *rem;   /* empty seats per row */
    long long *mx;    /* segment tree: max empty over range */
    long long *sum;   /* segment tree: total empty over range */
    int minr;         /* smallest row that may still hold empty seats */
} SeatReservation;

static void st_build(SeatReservation *s, int node, int lo, int hi) {
    if (lo == hi) { s->mx[node] = s->sum[node] = s->cap; return; }
    int mid = (lo + hi) / 2;
    st_build(s, 2 * node, lo, mid);
    st_build(s, 2 * node + 1, mid + 1, hi);
    s->mx[node] = s->mx[2*node] > s->mx[2*node+1] ? s->mx[2*node] : s->mx[2*node+1];
    s->sum[node] = s->sum[2*node] + s->sum[2*node+1];
}

static void st_update(SeatReservation *s, int node, int lo, int hi, int pos, long long val) {
    if (lo == hi) { s->mx[node] = s->sum[node] = val; return; }
    int mid = (lo + hi) / 2;
    if (pos <= mid) st_update(s, 2*node, lo, mid, pos, val);
    else st_update(s, 2*node+1, mid+1, hi, pos, val);
    s->mx[node] = s->mx[2*node] > s->mx[2*node+1] ? s->mx[2*node] : s->mx[2*node+1];
    s->sum[node] = s->sum[2*node] + s->sum[2*node+1];
}

static int st_first_ge(SeatReservation *s, int node, int lo, int hi, int limit, long long k) {
    if (lo > limit || s->mx[node] < k) return -1;
    if (lo == hi) return lo;
    int mid = (lo + hi) / 2;
    int left = st_first_ge(s, 2*node, lo, mid, limit, k);
    if (left != -1) return left;
    return st_first_ge(s, 2*node+1, mid+1, hi, limit, k);
}

static long long st_sum(SeatReservation *s, int node, int lo, int hi, int l, int r) {
    if (r < lo || hi < l) return 0;
    if (l <= lo && hi <= r) return s->sum[node];
    int mid = (lo + hi) / 2;
    return st_sum(s, 2*node, lo, mid, l, r) + st_sum(s, 2*node+1, mid+1, hi, l, r);
}

SeatReservation *seatReservationCreate(int rows, int cap) {
    SeatReservation *s = malloc(sizeof(SeatReservation));
    s->rows = rows;
    s->cap = cap;
    s->rem = malloc(sizeof(long long) * rows);
    for (int i = 0; i < rows; i++) s->rem[i] = cap;
    s->mx = calloc(4 * rows, sizeof(long long));
    s->sum = calloc(4 * rows, sizeof(long long));
    s->minr = 0;
    st_build(s, 1, 0, rows - 1);
    return s;
}

/* Returns malloc'd int[2] with *retSize=2, or NULL with *retSize=0 for []. */
int *seatReservationTogether(SeatReservation *s, int k, int limit, int *retSize) {
    int r = st_first_ge(s, 1, 0, s->rows - 1, limit, k);
    if (r == -1) { *retSize = 0; return NULL; }
    long long start = s->cap - s->rem[r];
    s->rem[r] -= k;
    st_update(s, 1, 0, s->rows - 1, r, s->rem[r]);
    int *ans = malloc(sizeof(int) * 2);
    ans[0] = r; ans[1] = (int)start;
    *retSize = 2;
    return ans;
}

int seatReservationSplit(SeatReservation *s, int k, int limit) {
    long long need = k;
    if (st_sum(s, 1, 0, s->rows - 1, 0, limit) < need) return 0;
    int i = s->minr;
    while (need > 0) {
        long long rem = s->rem[i];
        if (rem <= 0) { s->minr = ++i; continue; }
        if (rem <= need) {
            need -= rem;
            s->rem[i] = 0;
            st_update(s, 1, 0, s->rows - 1, i, 0);
            s->minr = ++i;
        } else {
            s->rem[i] = rem - need;
            st_update(s, 1, 0, s->rows - 1, i, s->rem[i]);
            need = 0;
        }
    }
    return 1;
}

void seatReservationFree(SeatReservation *s) {
    free(s->rem); free(s->mx); free(s->sum); free(s);
}

#### Follow-up
questions
How would you also support cancellations (freeing seats), and why does the simple forward pointer break once un-booking is allowed?
Can you answer together without descending twice — combining the max search and the update into a single traversal?
If cap were tiny (say <= 4) but rows huge, is there a lighter structure than a full segment tree?
How does the amortized argument for split change if limit could shrink the pointer's valid range on some calls?

