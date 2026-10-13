---
type: "practice"
difficulty: "Other"
---

# Rolling Event Counter over a Trailing 5-Minute Window

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queue.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a small data structure that tracks how many events have occurred within the most recent 5 minutes (i.e., the last 300 seconds).

The structure accepts a timeSec argument measured in seconds. You may assume calls arrive in chronological order, so the timeSec you receive is non-decreasing (monotonically increasing) across calls. Several events may be recorded at the very same second.

Implement a class RecentEventCounter with the following operations:

RecentEventCounter() — initialize an empty counter.
void record(int timeSec) — register that one event happened at second timeSec. Multiple events may share the same timeSec.
int countRecent(int timeSec) — return how many events happened within the trailing 300-second window ending at timeSec. Concretely, count every recorded event whose time t satisfies timeSec - t < 300 (an event exactly 300 seconds old has just fallen out of the window and is not counted).

Input/Output: the operations are applied in sequence. A constructor call returns nothing (null), record returns nothing (null), and countRecent returns an integer.

#### Cons
traints:

1 <= timeSec <= 2 * 10^9
All calls are made in chronological order (timeSec is monotonically non-decreasing).
At most 300 total calls will be made to record and countRecent combined.

Follow up: what if the number of events per second could be enormous? Does your design still use bounded memory?

#### Example
s & test cases
TEXT
- Input: ops = ["RecentEventCounter","record","record","record","countRecent","record","countRecent","countRecent"], args = [[],[1],[2],[3],[4],[300],[300],[301]] -> Output: [null,null,null,null,3,null,4,3]
  Why: at time 4 the events at 1,2,3 are all within 300s; at 300 the event at 1 still counts (300-1=299<300) giving 4; at 301 the event at 1 expires (301-1=300, not <300) leaving 3.
- Input: ops = ["RecentEventCounter","countRecent"], args = [[],[10]] -> Output: [null,0]
  Why: nothing has been recorded, so the window is empty.
- Input: ops = ["RecentEventCounter","record","record","record","countRecent"], args = [[],[1],[1],[1],[1]] -> Output: [null,null,null,null,3]
  Why: three events at the same timestamp are each counted.
- Input: ops = ["RecentEventCounter","record","countRecent"], args = [[],[5],[305]] -> Output: [null,null,0]
  Why: 305-5=300 is not < 300, so the event has just aged out at the window boundary.
- Input: ops = ["RecentEventCounter","record","countRecent"], args = [[],[5],[304]] -> Output: [null,null,1]
  Why: 304-5=299 < 300, so the event is still inside the window.
- Input: ops = ["RecentEventCounter","record","record","record","countRecent"], args = [[],[100],[200],[400],[400]] -> Output: [null,null,null,null,2]
  Why: the event at 100 expired (400-100=300, not <300); the events at 200 and 400 remain.

#### Approach

Use a circular buffer of exactly W = 300 buckets, indexing bucket timeSec % 300. Each bucket stores the timestamp it currently represents plus a count of events at that timestamp. On record, if the bucket already belongs to this timestamp, increment its count; otherwise the bucket holds a stale (older) timestamp that is now more than 300 seconds in the past, so overwrite it with the new timestamp and a count of 1 — a lazy, on-write expiration. On countRecent, scan all 300 buckets and add the counts of those whose stored timestamp is still within the window (timeSec - stored < 300). Because each second maps to a unique bucket within any 300-second span, this never double-counts and never needs unbounded storage. The alternative queue-of-timestamps design works too, but the fixed buffer is superior when the per-second hit rate is huge because it collapses same-second hits into a single counter.

Complexity

Time: record is O(1) (one modulo and a bucket update); countRecent is O(W) with W = 300, i.e. O(1) since the window size is a fixed constant. Space: O(W) = O(300) = O(1) — two fixed-size arrays regardless of how many events arrive. This is what answers the follow-up: memory stays bounded even if millions of events land in a single second, because same-second events merely increment one bucket's count.

Python
PYTHON
class RecentEventCounter:
    def __init__(self):
        self._window = 300
        self._timestamps = [0] * self._window  # timestamp owning each bucket
        self._counts = [0] * self._window      # events for that timestamp

    def record(self, time_sec: int) -> None:
        i = time_sec % self._window
        if self._timestamps[i] == time_sec:
            self._counts[i] += 1
        else:
            self._timestamps[i] = time_sec  # lazily reset a stale bucket
            self._counts[i] = 1

    def count_recent(self, time_sec: int) -> int:
        return sum(c for t, c in zip(self._timestamps, self._counts)
                   if time_sec - t < self._window)
JavaScript
JAVASCRIPT
class RecentEventCounter {

#### cons
tructor() {
    this.window = 300;
    this.timestamps = new Array(this.window).fill(0);
    this.counts = new Array(this.window).fill(0);
  }

  record(timeSec) {

#### cons
t i = timeSec % this.window;
    if (this.timestamps[i] === timeSec) {
      this.counts[i]++;
    } else {
      this.timestamps[i] = timeSec; // lazily reset a stale bucket
      this.counts[i] = 1;
    }
  }

  countRecent(timeSec) {
    let total = 0;
    for (let i = 0; i < this.window; i++) {
      if (timeSec - this.timestamps[i] < this.window) {
        total += this.counts[i];
      }
    }
    return total;
  }
}
Java
JAVA
class RecentEventCounter {
  private static final int WINDOW = 300;
  private final int[] timestamps = new int[WINDOW];
  private final int[] counts = new int[WINDOW];

  public RecentEventCounter() {}

  public void record(int timeSec) {
    final int i = timeSec % WINDOW;
    if (timestamps[i] == timeSec) {
      ++counts[i];
    } else {
      timestamps[i] = timeSec;  // lazily reset a stale bucket
      counts[i] = 1;
    }
  }

  public int countRecent(int timeSec) {
    int total = 0;
    for (int i = 0; i < WINDOW; ++i)
      if (timeSec - timestamps[i] < WINDOW)
        total += counts[i];
    return total;
  }
}
C++
CPP
#include <vector>
using namespace std;

class RecentEventCounter {
 public:
  RecentEventCounter() : timestamps(kWindow, 0), counts(kWindow, 0) {}

  void record(int timeSec) {

#### cons
t int i = timeSec % kWindow;
    if (timestamps[i] == timeSec) {
      ++counts[i];
    } else {
      timestamps[i] = timeSec;  // lazily reset a stale bucket
      counts[i] = 1;
    }
  }

  int countRecent(int timeSec) {
    int total = 0;
    for (int i = 0; i < kWindow; ++i)
      if (timeSec - timestamps[i] < kWindow)
        total += counts[i];
    return total;
  }

 private:
  static constexpr int kWindow = 300;
  vector<int> timestamps;
  vector<int> counts;
};
C
C
#include <stdlib.h>

#define WINDOW 300

typedef struct {
    int timestamps[WINDOW];  // timestamp currently owning each bucket
    int counts[WINDOW];      // events accumulated for that timestamp
} RecentEventCounter;

RecentEventCounter* recentEventCounterCreate() {
    // calloc zeroes both arrays; real timestamps are >= 1, so 0 means "empty".
    return (RecentEventCounter*)calloc(1, sizeof(RecentEventCounter));
}

void recentEventCounterRecord(RecentEventCounter* obj, int timeSec) {
    int i = timeSec % WINDOW;
    if (obj->timestamps[i] == timeSec) {
        obj->counts[i]++;
    } else {
        obj->timestamps[i] = timeSec;  // lazily overwrite the stale bucket
        obj->counts[i] = 1;
    }
}

int recentEventCounterCountRecent(RecentEventCounter* obj, int timeSec) {
    int total = 0;
    for (int i = 0; i < WINDOW; i++)
        if (timeSec - obj->timestamps[i] < WINDOW)
            total += obj->counts[i];
    return total;
}

void recentEventCounterFree(RecentEventCounter* obj) {
    free(obj);
}

#### Follow-up
questions
If the per-second event rate is enormous, does the design still hold? (Yes — same-second hits collapse into one bucket, so memory stays O(300).)
Generalize the window to an arbitrary length W passed at construction, or a very large window such as 24 hours — compare the circular-buffer and timestamp-queue trade-offs.
Make record and countRecent safe under many concurrent writer threads.
What if timestamps could arrive slightly out of order rather than strictly non-decreasing?

