---
type: "practice"
difficulty: "Other"
---

# Wrap a Function So It Executes At Most Once

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Closures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Closures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a function target. Implement a higher-order function once(target) that returns a new function guarded. The returned function must behave as follows:

The first time guarded is invoked, it must call target with exactly the arguments it received and return whatever target returns.
Every subsequent invocation of guarded must not call target at all, and must return the language's "no value" sentinel (undefined in JavaScript, None in Python, null/an absent value in the statically typed variants).

In other words, no matter how many times the wrapped function is called, the underlying target is executed a maximum of one time, and only the very first call ever produces target's result.

To make the behavior testable across languages, assume target takes a list of integers and returns an integer. You are given the list of argument-lists to feed to guarded in order; report, for each invocation, whether a value was produced and, if so, what it was.

Input

target: a pure function mapping a list of integers to a single integer.
calls: an ordered list of argument-lists; calls[i] is the list of integers passed on the i-th invocation of guarded.

Output

For each invocation, either the produced integer (first call only) or the "no value" sentinel (all later calls).

#### Cons
traints

1 <= calls.length <= 12
1 <= calls[i].length <= 100
Each integer fits in a 32-bit signed range.
target is deterministic and has no side effects other than being observed by the caller.

#### Example
s & test cases
TEXT
Input: target = (list) -> sum(list); calls = [[4, 5, 6], [10, 20, 30]] -> Output: [6+... = 15, undefined]
Why: First call returns sum(4,5,6)=15; second call is suppressed and returns the no-value sentinel.

Input: target = (list) -> product(list); calls = [[3, 3, 3], [2, 2], [9]] -> Output: [27, undefined, undefined]
Why: Only the first call runs target (3*3*3=27); the two later calls return nothing.

Input: target = (list) -> sum(list); calls = [[7]] -> Output: [7]
Why: A single invocation always runs target and returns its result (7).

Input: target = (list) -> sum(list); calls = [[-5, 5], [1], [1000000, 1000000]] -> Output: [0, undefined, undefined]
Why: First call handles negatives correctly (-5+5=0); later calls are suppressed regardless of their arguments.

Input: target = (list) -> max(list); calls = [[-3, -7, -1], [100]] -> Output: [-1, undefined]
Why: First call evaluates target on all-negative input (max = -1); the second call returns the no-value sentinel.

#### Approach

This is a closure / higher-order-function pattern. once captures a single private boolean flag in the enclosing scope and returns an inner function that closes over it. On each invocation the inner function inspects the flag: if it is still unset, it flips the flag, forwards the received arguments to target, and returns target's result; if the flag is already set, it short-circuits and returns the "no value" sentinel without ever touching target. Argument forwarding (...args / apply / *args) makes the wrapper transparent to any signature. In languages without closures (C), the captured flag is stored explicitly in a struct that stands in for the closure's environment.

Complexity

Time: O(1) overhead per invocation on top of the cost of target (which runs at most once). Space: O(1) — a single boolean flag captured in the closure. Justification: the wrapper only checks/flips one flag; it stores no growing state.

Python
PYTHON
def once(target):
    """Return a wrapper that calls `target` at most one time.
    First call returns target's result; every later call returns None."""
    called = False
    def guarded(*args, **kwargs):
        nonlocal called
        if called:
            return None            # later calls: target is not invoked
        called = True
        return target(*args, **kwargs)  # first call: run target once
    return guarded
JavaScript
JAVASCRIPT
/**
 * @param {Function} target
 * @return {Function} a wrapper that runs `target` at most once
 */
function once(target) {
  let called = false;
  return function (...args) {
    if (called) return undefined;   // later calls: target not invoked
    called = true;
    return target.apply(this, args); // first call: run target once
  };
}
Java
JAVA
import java.util.function.Function;

class Solution {
    // once(target) returns a Function that invokes target at most once.
    // Integer null plays the role of the "no value" sentinel.
    public static Function<int[], Integer> once(Function<int[], Integer> target) {
        boolean[] called = { false };   // one-element array = mutable captured state
        return (args) -> {
            if (called[0]) {
                return null;            // subsequent calls: target not invoked
            }
            called[0] = true;
            return target.apply(args);  // first call: run target once
        };
    }
}
C++
CPP
#include <functional>
#include <optional>
#include <vector>
#include <memory>
using namespace std;

// once(target) returns a callable that runs target at most one time.
// A shared flag lives in the closure so the state persists across calls.
function<optional<int>(const vector<int>&)>
once(function<int(const vector<int>&)> target) {
    auto called = make_shared<bool>(false);
    return [target, called] (const vector<int>& args) -> optional<int> {
        if (*called) {
            return nullopt;              // later calls: no value, target skipped
        }
        *called = true;
        return target(args);             // first call: run target once
    };
}
C
C
#include <stdbool.h>
#include <stddef.h>

/* C has no closures, so the "captured state" is held explicitly in a struct.
   Once acts as the wrapper: it remembers the target and whether it has fired. */

typedef int (*IntListFn)(const int *args, int n);

typedef struct {
    IntListFn target;
    bool called;
} Once;

/* once(target) -> build the guarded wrapper */
void once_init(Once *g, IntListFn target) {
    g->target = target;
    g->called = false;
}

/* Invoke the guarded function.
   Returns true and writes the result to *out on the FIRST call only;
   returns false (the "no value" sentinel) on every later call. */
bool once_call(Once *g, const int *args, int n, int *out) {
    if (g->called) {
        return false;          /* subsequent calls: target NOT invoked */
    }
    g->called = true;
    *out = g->target(args, n); /* first call: run target once */
    return true;
}

#### Follow-up
questions
Generalize to "at most N times" — return the sentinel only after the N-th successful call.
Make the wrapper cache the first result and return that cached value on every later call instead of the sentinel (memoized-once).
Make it thread-safe so concurrent callers still trigger target exactly once (e.g., compare-and-swap on the flag or a call-once primitive).
Preserve the original function's this binding and support asynchronous target that returns a promise.

