---
type: "practice"
difficulty: "Other"
---

# Delayed Completion Routine

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrency](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Implement a routine that pauses for a requested amount of time and then completes.

Given a positive integer delayMs, write a routine that suspends for approximately delayMs milliseconds and then finishes (it may finish with any result value, or none). The routine must not complete earlier than delayMs milliseconds after it begins; a small deviation above the requested duration is acceptable (real timers are not exact).

In an asynchronous or concurrent setting, the routine should perform the wait without busy looping — it should yield the processor for the duration rather than spinning — so that other work can proceed while it waits. It then completes once the delay has elapsed.

Input / Output. Input is a single positive integer delayMs. The observable "output" is timing: measuring the wall-clock time from the moment the routine starts to the moment it completes should give a value no less than delayMs (within normal scheduler tolerance).

#### Cons
traints.

1 <= delayMs <= 1000 (milliseconds).

#### Example
s & test cases
TEXT
Input: delayMs = 150 -> Output: completes after ~150 ms
Why: typical case; elapsed time measured at completion is about 150 ms, not less.

Input: delayMs = 300 -> Output: completes after ~300 ms
Why: a longer wait; roughly doubles the elapsed time versus the 150 ms case.

Input: delayMs = 1 -> Output: completes after ~1 ms
Why: lower bound of the constraint; the shortest possible pause.

Input: delayMs = 1000 -> Output: completes after ~1000 ms
Why: upper bound; the longest allowed pause.

Input: delayMs = 500 -> Output: completes after ~500 ms (never before)
Why: confirms the routine must not finish early — completion time is >= delayMs.

#### Approach

The pattern is a non-blocking (cooperative) delay via the platform's timer facility. Rather than spin in a loop checking the clock (which wastes CPU), the routine hands the delay to the scheduler: setTimeout + a Promise in JavaScript, asyncio.sleep in Python (both yield to the event loop), and a thread-suspending sleep (nanosleep, sleep_for, Thread.sleep) in C/C++/Java. In every case the routine is guaranteed not to complete before delayMs has elapsed, other work may proceed meanwhile, and completion carries any (or no) result value.

Complexity

Time: O(1) in terms of computational work — the routine does a constant amount of setup and then simply waits; the wall-clock duration is delayMs by design, not algorithmic cost. Space: O(1) — only a single timer/handle (or timespec) is held.

Python
PYTHON
import asyncio

async def sleep_millis(delay_ms):
    """Asynchronously pause for delay_ms milliseconds, then complete.

    await asyncio.sleep yields to the event loop (no busy waiting), so other
    coroutines run while this one waits."""
    await asyncio.sleep(delay_ms / 1000.0)

async def _demo():
    import time
    start = time.perf_counter()
    await sleep_millis(150)
    elapsed = (time.perf_counter() - start) * 1000
    print(f"elapsed ~{elapsed:.0f} ms")  # ~150

if __name__ == "__main__":
    asyncio.run(_demo())
JavaScript
JS
// Returns a promise that resolves after delayMs milliseconds.
// setTimeout schedules the resolution without blocking the event loop.
function sleep(delayMs) {
  return new Promise((resolve) => setTimeout(resolve, delayMs));
}

// Demonstration
(async () => {

#### cons
t start = Date.now();
  await sleep(150);

#### cons
ole.log(`elapsed ~${Date.now() - start} ms`); // ~150
})();
Java
JAVA
public class Solution {
    // Suspend the current thread for delayMs milliseconds; Thread.sleep yields the CPU.
    public void sleepMillis(long delayMs) throws InterruptedException {
        Thread.sleep(delayMs);
    }

    public static void main(String[] args) throws InterruptedException {

#### Solution
s = new Solution();
        long start = System.currentTimeMillis();
        s.sleepMillis(150);
        long elapsed = System.currentTimeMillis() - start;
        System.out.println("elapsed ~" + elapsed + " ms"); // ~150
    }
}
C++
CPP
#include <chrono>
#include <thread>
#include <iostream>

// Sleep this thread for delayMs milliseconds; sleep_for yields, no busy loop.
void sleepMillis(int delayMs) {
    std::this_thread::sleep_for(std::chrono::milliseconds(delayMs));
}

int main() {
    auto start = std::chrono::steady_clock::now();
    sleepMillis(150);
    auto end = std::chrono::steady_clock::now();
    auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count();
    std::cout << "elapsed ~" << ms << " ms\n"; // ~150
    return 0;
}
C
C
#include <stdio.h>
#include <time.h>

/* Pause the current thread for delayMs milliseconds without busy-waiting.
   nanosleep yields the CPU for the requested span. */
void sleepMillis(int delayMs) {
    struct timespec ts;
    ts.tv_sec  =  delayMs / 1000;
    ts.tv_nsec = (long)(delayMs % 1000) * 1000000L; /* ms -> ns */
    nanosleep(&ts, NULL);
}

int main(void) {
    struct timespec a, b;
    clock_gettime(CLOCK_MONOTONIC, &a);
    sleepMillis(150);
    clock_gettime(CLOCK_MONOTONIC, &b);
    double elapsed = (b.tv_sec - a.tv_sec) * 1000.0
                   + (b.tv_nsec - a.tv_nsec) / 1e6;
    printf("elapsed ~%.0f ms\n", elapsed); /* ~150 */
    return 0;
}

#### Follow-up
questions
Make the pending delay cancellable so it can be aborted before it fires.
Have the routine complete with a caller-supplied value instead of nothing.
Build a timeout wrapper: run another async operation but reject if it exceeds delayMs.
Measure and report the actual timer drift versus the requested delayMs.

