---
type: "practice"
difficulty: "Other"
---

# Enforce a Time Budget on a Long-Running Task

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrency](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a task task that performs some work and eventually produces a result — it may run for a while and its outcome is only available once it finishes (think of an asynchronous or blocking operation). You are also given a time budget limitMs in milliseconds.

Return a new wrapped function that accepts the same arguments task expects and behaves as follows:

If task finishes and produces its result within limitMs milliseconds, the wrapper completes successfully with that same result.
If task is still running when limitMs milliseconds have elapsed, the wrapper fails with the exact string "Time Limit Exceeded".
If task fails on its own (raises/throws/rejects) before the limit is reached, the wrapper fails with that original error, unchanged.

In other words: race the task's completion against a timer of length limitMs, and report whichever settles first. The wrapper must not alter a successful result or a genuine task error; it only substitutes the "Time Limit Exceeded" failure when the timer wins.

Input:

task: a function that, given its arguments, eventually yields a result or fails.
limitMs: the time budget in milliseconds.

Output:

A wrapped function with the same call signature as task, obeying the three rules above.

#### Cons
traints:

0 <= number of arguments passed to task <= 10
0 <= limitMs <= 1000
task eventually produces a result or reports an error.

#### Example
s & test cases
TEXT
Input: task runs ~100ms then returns n*n; args = [6]; limitMs = 40  ->  Output: rejected "Time Limit Exceeded" at ~40ms
Why: the task is slower than the budget, so the timer wins and the sentinel error is reported.

Input: task runs ~80ms then returns n*n; args = [6]; limitMs = 200  ->  Output: resolved 36 at ~80ms
Why: the task finishes comfortably inside the budget, so its real result is returned.

Input: task runs ~90ms then returns a+b; args = [3, 4]; limitMs = 200  ->  Output: resolved 7 at ~90ms
Why: multi-argument task inside the budget returns its own computed value.

Input: task immediately fails with "boom"; args = []; limitMs = 500  ->  Output: rejected "boom" at ~0ms
Why: the task reports its own error before the limit, so that original error propagates unchanged (not overridden by the timeout message).

Input: task runs ~50ms then returns 1; args = []; limitMs = 0  ->  Output: rejected "Time Limit Exceeded" at ~0ms
Why: edge case — a zero budget means anything not already complete is immediately over the limit.

#### Approach

The pattern is concurrency with a timeout (a race). Start the task on a separate execution context — an event-loop promise in JavaScript, or a worker thread in the blocking languages — so it can make progress independently. Simultaneously arm a timer for limitMs. Whichever settles first decides the outcome: if the task's completion wins, forward its result (or its own error) unchanged; if the timer wins, produce the sentinel failure "Time Limit Exceeded". The key subtleties are (1) not overwriting a genuine task error with the timeout message, and (2) cancelling/clearing the loser (clear the timer when the task finishes early, or cancel the worker when the timeout fires) so resources are released.

Complexity

Time: The wrapper adds only O(1) control overhead on top of the task; the wall-clock time it may wait is bounded by min(task's own duration, limitMs). It does not add CPU work proportional to any input size. Space: O(1) auxiliary — one timer plus one extra execution context (thread or microtask) to run the task concurrently.

Python
PYTHON
import concurrent.futures
from typing import Any, Callable

def time_limit(task: Callable[..., Any], limit_ms: int) -> Callable[..., Any]:
    """Return a wrapper that runs `task` but aborts with 'Time Limit Exceeded'
    if it takes longer than limit_ms milliseconds. A genuine error raised by
    `task` before the limit propagates unchanged."""
    def wrapped(*args):
        with concurrent.futures.ThreadPoolExecutor(max_workers=1) as pool:
            future = pool.submit(task, *args)
            try:
                return future.result(timeout=limit_ms / 1000.0)
            except concurrent.futures.TimeoutError:
                raise Exception("Time Limit Exceeded")
            # any other exception from task is re-raised by future.result()
    return wrapped
JavaScript
JAVASCRIPT
/**
 * @param {Function} task      an async function returning a promise
 * @param {number}   limitMs   time budget in milliseconds
 * @return {Function}          a time-limited async version of task
 */
function timeLimit(task, limitMs) {
  return async function (...args) {
    return new Promise((resolve, reject) => {

#### cons
t timer = setTimeout(() => reject("Time Limit Exceeded"), limitMs);
      task(...args)
        .then((value) => { clearTimeout(timer); resolve(value); })
        .catch((err)  => { clearTimeout(timer); reject(err); });
    });
  };
}
Java
JAVA
import java.util.concurrent.*;
import java.util.function.Supplier;

class Solution {
    /**
     * Runs task but fails with "Time Limit Exceeded" if it exceeds limitMs.
     * If task fails on its own first, that error's message propagates.
     */
    public <T> T timeLimit(Supplier<T> task, long limitMs) throws Exception {
        ExecutorService pool = Executors.newSingleThreadExecutor();
        Future<T> future = pool.submit(task::get);
        try {
            return future.get(limitMs, TimeUnit.MILLISECONDS);
        } catch (TimeoutException e) {
            future.cancel(true);
            throw new Exception("Time Limit Exceeded");
        } catch (ExecutionException e) {
            Throwable cause = e.getCause();               // task's own failure
            throw new Exception(cause == null ? e.getMessage() : cause.getMessage());
        } finally {
            pool.shutdownNow();
        }
    }
}
C++
CPP
#include <future>
#include <chrono>
#include <functional>
#include <stdexcept>
using namespace std;

class Solution {
public:
    // Runs task; throws runtime_error("Time Limit Exceeded") if it does not
    // finish within limitMs. Any exception thrown by task propagates unchanged.
    template <typename T>
    T timeLimit(const function<T()>& task, long limitMs) {
        future<T> fut = async(launch::async, task);
        if (fut.wait_for(chrono::milliseconds(limitMs)) == future_status::timeout) {
            throw runtime_error("Time Limit Exceeded");
        }
        return fut.get();  // re-throws task's own exception if it failed
    }
};
C
C
#define _GNU_SOURCE
#include <pthread.h>
#include <time.h>
#include <errno.h>

/*
 * Runs a task on a worker thread and waits at most limit_ms for it.
 * The task writes its result via `out` and returns 0 on success, non-zero to
 * signal its own error. time_limit returns:
 *    0 -> success (result stored in *result_out)
 *    1 -> "Time Limit Exceeded" (task did not finish in time)
 *    2 -> task reported its own error
 */
typedef struct {
    int  (*task)(void* args, long* out); /* 0 = ok (writes *out), non-0 = own error */
    void* args;
    long  result;
    int   status;                        /* 0 ok, non-0 task error */
} TaskCtx;

static void* run_task(void* p) {
    TaskCtx* c = (TaskCtx*)p;
    c->status = c->task(c->args, &c->result);
    return NULL;
}

int time_limit(TaskCtx* ctx, long limit_ms, long* result_out) {
    pthread_t th;
    pthread_create(&th, NULL, run_task, ctx);

    struct timespec deadline;
    clock_gettime(CLOCK_REALTIME, &deadline);
    deadline.tv_sec  += limit_ms / 1000;
    deadline.tv_nsec += (limit_ms % 1000) * 1000000L;
    if (deadline.tv_nsec >= 1000000000L) {
        deadline.tv_sec  += 1;
        deadline.tv_nsec -= 1000000000L;
    }

    int rc = pthread_timedjoin_np(th, NULL, &deadline);
    if (rc == ETIMEDOUT) {
        pthread_cancel(th);          /* stop the worker; free its slot */
        pthread_join(th, NULL);
        return 1;                    /* Time Limit Exceeded */
    }
    if (ctx->status != 0) return 2;  /* task's own error */
    *result_out = ctx->result;
    return 0;                        /* success */
}

#### Follow-up
questions
On timeout, must you actually stop/cancel the underlying task (free its resources), or is merely reporting the failure enough — and how does that differ across languages?
Support limitMs = 0 meaning "no limit at all" versus "zero budget"; how would you distinguish the two contracts?
Add automatic retries: on a timeout, re-run the task up to k times before finally failing.
Guarantee the timer is always cleared so the process or event loop can exit promptly when the task finishes early — what leaks if you forget?

