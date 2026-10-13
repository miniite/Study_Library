---
type: "practice"
difficulty: "Other"
---

# Reservation Ledger Without Triple Overlap

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a reservation ledger that accepts time-range bookings but never allows three bookings to overlap at the same instant. A booking is a half-open interval [start, end), meaning it covers every real number x with start <= x < end. Two or more bookings overlap when there is at least one instant common to all of them.

A triple overlap occurs when three bookings all share some common instant. The ledger may allow a double overlap (two bookings sharing time), but it must reject any booking that would cause a triple overlap.

Implement a class ReservationLedger with:

ReservationLedger() — creates an empty ledger.
boolean book(int start, int end) — if adding the interval [start, end) keeps the ledger free of any triple overlap, add it and return true; otherwise do not add it and return false.

Because the intervals are half-open, two bookings that only touch at a boundary (one ends exactly where the other begins) do not overlap.

Input: a sequence of book(start, end) calls on one ledger instance.

Output: for each call, a boolean — true if the booking was accepted (and stored), false if it was rejected (and not stored). A rejected booking must leave the ledger exactly as it was.

#### Cons
traints:

0 <= start < end <= 10^9.
At most 1000 calls are made to book on a single ledger.

#### Example
s & test cases
TEXT
Input: book(30,45), book(70,90), book(35,60), book(40,50), book(45,55), book(58,75) -> Output: true, true, true, false, true, true
Why: [35,60) double-books [35,45) with [30,45); [40,50) would make [40,45) a triple so it is rejected; [45,55) and [58,75) each stay at a double.

Input: book(0,10), book(0,10), book(0,10) -> Output: true, true, false
Why: the first two create a double over [0,10); the third would make it a triple and is rejected.

Input: book(0,5), book(5,10), book(0,5) -> Output: true, true, true
Why: [0,5) and [5,10) only touch at 5 (excluded by the half-open range) so they do not overlap; the second [0,5) merely double-books [0,5).

Input: book(0,20), book(10,30), book(5,15), book(25,40) -> Output: true, true, false, true
Why: [5,15) would triple-book the already double-booked [10,20) and is rejected; because it was not stored, the later [25,40) still fits as a double with [10,30).

Input: book(0,1000000000) -> Output: true
Why: a single booking over the maximum range is always accepted since no overlap is possible yet (minimum / boundary case).

#### Approach

Keep two lists: bookings, every accepted interval, and overlaps, every region that is currently covered by exactly two bookings. To add [start, end): first check it against overlaps — if it intersects any already double-booked region, a third booking would cover that instant, so reject. Otherwise it is safe; compute its intersection with each existing booking and record each non-empty intersection as a new double-booked region, then store the interval itself. Two half-open intervals [a,b) and [c,d) intersect exactly when max(a,c) < min(b,d), which also correctly treats boundary-touching intervals as non-overlapping. A rejected booking returns before mutating either list, so the ledger is unchanged. The underlying pattern is interval-overlap bookkeeping (a lightweight form of a difference-array / sweep-line count capped at 2).

Complexity

Let n be the number of accepted bookings and m the number of stored double-booked regions. Each book scans the overlap regions (O(m)) and, if accepted, scans all prior bookings (O(n)). Across n calls this is O(n^2) work to build overlaps plus the overlap checks; m can grow to O(n^2) in the worst case, so the total is between O(n^2) and O(n^3) worst case — comfortably within limits for n <= 1000. Space is O(n + m), i.e. O(n^2) worst case. An alternative using an ordered map of boundary deltas (sweep the running count, reject if it ever exceeds 2) gives O(n log n) per call.

Python
PYTHON
class ReservationLedger:
    def __init__(self):
        self.bookings = []   # all accepted intervals
        self.overlaps = []   # regions currently double-booked

    def book(self, start: int, end: int) -> bool:
        for s, e in self.overlaps:
            if max(start, s) < min(end, e):
                return False                      # would create a triple overlap
        for s, e in self.bookings:
            lo, hi = max(start, s), min(end, e)
            if lo < hi:
                self.overlaps.append((lo, hi))    # new double-booked region
        self.bookings.append((start, end))
        return True
JavaScript
JAVASCRIPT
class ReservationLedger {

#### cons
tructor() {
        this.bookings = [];   // all accepted intervals
        this.overlaps = [];   // regions currently double-booked
    }

    book(start, end) {
        for (const [s, e] of this.overlaps) {
            if (Math.max(start, s) < Math.min(end, e)) return false;  // would triple-book
        }
        for (const [s, e] of this.bookings) {

#### cons
t lo = Math.max(start, s), hi = Math.min(end, e);
            if (lo < hi) this.overlaps.push([lo, hi]);                // new double-booked region
        }
        this.bookings.push([start, end]);
        return true;
    }
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class ReservationLedger {
    private final List<int[]> bookings = new ArrayList<>();  // all accepted intervals
    private final List<int[]> overlaps = new ArrayList<>();  // regions currently double-booked

    public ReservationLedger() {}

    public boolean book(int start, int end) {
        for (int[] o : overlaps)
            if (Math.max(start, o[0]) < Math.min(end, o[1]))
                return false;                       // would create a triple overlap
        for (int[] b : bookings) {
            int s = Math.max(start, b[0]);
            int e = Math.min(end, b[1]);
            if (s < e) overlaps.add(new int[]{s, e});   // new double-booked region
        }
        bookings.add(new int[]{start, end});
        return true;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class ReservationLedger {
public:
    bool book(int start, int end) {
        for (const auto& [s, e] : overlaps)
            if (max(start, s) < min(end, e))
                return false;                       // would create a triple overlap
        for (const auto& [s, e] : bookings) {
            int ss = max(start, s), ee = min(end, e);
            if (ss < ee) overlaps.push_back({ss, ee});   // new double-booked region
        }
        bookings.push_back({start, end});
        return true;
    }
private:
    vector<pair<int,int>> bookings;   // all accepted intervals
    vector<pair<int,int>> overlaps;   // regions currently double-booked
};
C
C
#include <stdbool.h>
#include <stdlib.h>

typedef struct {
    int (*bookings)[2];
    int bookingCount, bookingCap;
    int (*overlaps)[2];        /* regions already double-booked */
    int overlapCount, overlapCap;
} ReservationLedger;

ReservationLedger* reservationLedgerCreate(void) {
    ReservationLedger* r = (ReservationLedger*)malloc(sizeof(ReservationLedger));
    r->bookingCap = 16; r->bookingCount = 0;
    r->bookings = (int(*)[2])malloc(sizeof(int[2]) * r->bookingCap);
    r->overlapCap = 16; r->overlapCount = 0;
    r->overlaps = (int(*)[2])malloc(sizeof(int[2]) * r->overlapCap);
    return r;
}

static int imax(int a, int b) { return a > b ? a : b; }
static int imin(int a, int b) { return a < b ? a : b; }

bool reservationLedgerBook(ReservationLedger* r, int start, int end) {
    for (int i = 0; i < r->overlapCount; ++i)
        if (imax(start, r->overlaps[i][0]) < imin(end, r->overlaps[i][1]))
            return false;                 /* hitting a double-booked region => triple */

    for (int i = 0; i < r->bookingCount; ++i) {
        int s = imax(start, r->bookings[i][0]);
        int e = imin(end, r->bookings[i][1]);
        if (s < e) {                      /* new double-booked region */
            if (r->overlapCount == r->overlapCap) {
                r->overlapCap *= 2;
                r->overlaps = (int(*)[2])realloc(r->overlaps, sizeof(int[2]) * r->overlapCap);
            }
            r->overlaps[r->overlapCount][0] = s;
            r->overlaps[r->overlapCount][1] = e;
            r->overlapCount++;
        }
    }

    if (r->bookingCount == r->bookingCap) {
        r->bookingCap *= 2;
        r->bookings = (int(*)[2])realloc(r->bookings, sizeof(int[2]) * r->bookingCap);
    }
    r->bookings[r->bookingCount][0] = start;
    r->bookings[r->bookingCount][1] = end;
    r->bookingCount++;
    return true;
}

void reservationLedgerFree(ReservationLedger* r) {
    free(r->bookings);
    free(r->overlaps);
    free(r);
}

#### Follow-up
questions
Generalize to "no k-fold overlap": allow up to k-1 concurrent bookings and reject the k-th. How does the two-list idea extend (hint: a boundary-count map)?
Support cancelling a previously accepted booking and keeping the double-booked regions consistent.
After each insertion, report the maximum number of bookings overlapping at any single instant.
Reduce each book to O(log n) using an ordered map / balanced BST of +1/-1 boundary deltas instead of scanning all intervals.

