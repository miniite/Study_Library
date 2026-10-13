---
type: "practice"
difficulty: "Other"
---

# Sliding-Window Average of a Number Stream

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queue.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Numbers arrive one at a time in a continuous stream. You are asked to maintain the average of the most recent k values seen so far, where k is a fixed window size chosen when the stream starts. Every time a new number arrives, you must report the average of the last k numbers; if fewer than k numbers have arrived yet, report the average of all numbers seen so far.

Design a class StreamAverage that supports the following operations:

StreamAverage(int windowSize) — creates the object with the window size windowSize (the maximum number of most-recent values that participate in each average).
double next(int value) — records the incoming integer value and returns the average (as a floating-point number) of the last windowSize values of the stream, or of all values received so far if fewer than windowSize have arrived.

Each call to next reflects the value it just received. Once more than windowSize values have been received, the oldest value falls out of the window and no longer contributes to the average.

Input: a sequence of operations: one constructor call with the window size, followed by any number of next calls, each carrying one integer.

Output: for each next call, the current sliding-window average as a double.

#### Cons
traints:

1 <= windowSize <= 1000
-10^5 <= value <= 10^5
At most 10^4 calls will be made to next.

#### Example
s & test cases
TEXT
Input: StreamAverage(3); next(1) -> 1.0; next(10) -> 5.5; next(3) -> 4.66667; next(5) -> 6.0
Why: with only one value the average is 1/1; then (1+10)/2; then (1+10+3)/3; on the fourth call the window is full so the oldest value 1 is evicted and the average is (10+3+5)/3.

Input: StreamAverage(1); next(4) -> 4.0; next(9) -> 9.0
Why: a window of size one always holds just the latest value, so each average equals the number just received.

Input: StreamAverage(2); next(-2) -> -2.0; next(2) -> 0.0; next(6) -> 4.0
Why: negative values are averaged normally; on the third call the window is full so -2 is dropped and the average is (2+6)/2 = 4.0.

Input: StreamAverage(5); next(7) -> 7.0
Why: fewer numbers have arrived than the window size, so the average uses only the single value present.

Input: StreamAverage(3); next(100000) -> 100000.0; next(-100000) -> 0.0; next(0) -> 0.0
Why: extreme in-range magnitudes cancel correctly and the running sum stays exact.

#### Approach

Keep a first-in-first-out queue holding just the values currently inside the window, plus a running sum of those values. On each next: if the queue is already full (its size equals the window size), subtract the oldest value from sum and remove it from the front; then push the new value onto the back and add it to sum. The answer is sum divided by the current queue size. Maintaining the running sum incrementally avoids re-adding the whole window each time, giving O(1) work per element — the classic queue-backed sliding-window aggregation pattern.

Complexity

Time: O(1) per next call — a single eviction, a single insertion, and a constant-work update of the running sum, all amortized constant (the C version uses a fixed circular buffer, the JS version an advancing head index, the others an O(1) queue). Space: O(k) where k is the window size, since only the live window is retained.

Python
PYTHON
from collections import deque

class StreamAverage:
    def __init__(self, windowSize: int):
        self.capacity = windowSize
        self.window = deque()
        self.sum = 0

    def next(self, value: int) -> float:
        if len(self.window) == self.capacity:
            self.sum -= self.window.popleft()
        self.window.append(value)
        self.sum += value
        return self.sum / len(self.window)
JavaScript
JAVASCRIPT
class StreamAverage {

#### cons
tructor(windowSize) {
        this.capacity = windowSize;
        this.window = [];
        this.head = 0;     // index of the oldest live element
        this.sum = 0;
    }

    next(value) {
        if (this.window.length - this.head === this.capacity) {
            this.sum -= this.window[this.head++];
        }
        this.window.push(value);
        this.sum += value;
        return this.sum / (this.window.length - this.head);
    }
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class StreamAverage {
    private final int capacity;
    private double sum = 0;
    private final Deque<Integer> window = new ArrayDeque<>();

    public StreamAverage(int windowSize) {
        this.capacity = windowSize;
    }

    public double next(int value) {
        if (window.size() == capacity) {
            sum -= window.poll();
        }
        window.offer(value);
        sum += value;
        return sum / window.size();
    }
}
C++
CPP
#include <queue>
using namespace std;

class StreamAverage {
public:
    StreamAverage(int windowSize) : capacity(windowSize) {}

    double next(int value) {
        if ((int)window.size() == capacity) {
            sum -= window.front();
            window.pop();
        }
        window.push(value);
        sum += value;
        return sum / window.size();
    }

private:
    int capacity;
    double sum = 0;
    queue<int> window;
};
C
C
#include <stdlib.h>

typedef struct {
    int* buf;        /* circular buffer of the live window */
    int capacity;    /* window size */
    int count;       /* how many live values are currently stored */
    int head;        /* index of the oldest live value */
    long long sum;   /* running sum of the live values */
} StreamAverage;

StreamAverage* streamAverageCreate(int windowSize) {
    StreamAverage* obj = (StreamAverage*)malloc(sizeof(StreamAverage));
    obj->buf = (int*)malloc(sizeof(int) * windowSize);
    obj->capacity = windowSize;
    obj->count = 0;
    obj->head = 0;
    obj->sum = 0;
    return obj;
}

double streamAverageNext(StreamAverage* obj, int value) {
    if (obj->count == obj->capacity) {
        obj->sum -= obj->buf[obj->head];
        obj->head = (obj->head + 1) % obj->capacity;
        obj->count--;
    }
    int tail = (obj->head + obj->count) % obj->capacity;
    obj->buf[tail] = value;
    obj->sum += value;
    obj->count++;
    return (double)obj->sum / obj->count;
}

void streamAverageFree(StreamAverage* obj) {
    free(obj->buf);
    free(obj);
}

#### Follow-up
questions
Support a very large window (millions of values) — how do you keep O(1) memory with just a running sum, and how do you guard against floating-point drift or overflow?
Return the moving median of the window instead of the moving average.
Make next thread-safe so multiple producers can push concurrently.
Switch from a fixed count to a time-based window: each value carries a timestamp and expires after a fixed duration rather than after a fixed number of arrivals.

