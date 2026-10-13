---
type: "practice"
difficulty: "Other"
---

# Auto-Incrementing Value Generator

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Closures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Closures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are asked to build a small "value generator" factory.

Given an integer start, construct a stateful generator. Each time the generator is invoked (with no arguments), it returns the next value of an ascending integer sequence that begins at start:

The first invocation returns start.
The second invocation returns start + 1.
The third invocation returns start + 2.
... and so on, each subsequent invocation returning exactly one more than the value it returned on the previous invocation.

The generator must keep its own private counting state between invocations — calling it repeatedly walks the sequence forward. Two generators produced from separate calls to the factory must be completely independent of each other.

Input / Output. Conceptually you are given start and a list of invocation events (each event simply means "call the generator once"). You must return the list of values produced, in the order the generator was called. If there are no invocation events, the result is an empty list.

#### Cons
traints.

-1000 <= start <= 1000
0 <= number_of_invocations <= 1000
Every invocation event is a plain "call the generator" action (it carries no argument).

#### Example
s & test cases
TEXT
Input: start = 7, invocations = 3 -> Output: [7, 8, 9]
Why: first call returns start, each later call adds exactly one.

Input: start = -3, invocations = 5 -> Output: [-3, -2, -1, 0, 1]
Why: negative start; the sequence walks up through zero into positives.

Input: start = 0, invocations = 0 -> Output: []
Why: with zero invocations the generator is never called, so nothing is produced.

Input: start = 1000, invocations = 1 -> Output: [1000]
Why: a single call returns start unchanged (upper bound of start).

Input: start = -1000, invocations = 2 -> Output: [-1000, -999]
Why: lower bound of start; state advances by one on the second call.

#### Approach

The core pattern is state encapsulation via a closure (higher-order function). The factory captures a private current variable initialized to start and returns a small function that, on every call, returns current and then post-increments it. Because the variable lives in the returned function's own scope, the count survives between calls and stays isolated from any other generator. In languages without closures (C) the identical behavior is achieved by putting the state in a struct that is threaded through each call; in Java the mutable state is boxed in a one-element array so the lambda can update it.

Complexity

Time: O(1) per invocation — each call just reads and increments a single integer. Space: O(1) — the generator holds exactly one integer of private state, independent of how many times it is called.

Python
PYTHON
def create_counter(start):
    """Return a closure that yields start, start+1, start+2, ... on each call."""
    current = start

    def counter():
        nonlocal current
        value = current
        current += 1          # return current value, then advance
        return value

    return counter

if __name__ == "__main__":
    counter = create_counter(7)
    print([counter() for _ in range(3)])  # [7, 8, 9]
JavaScript
JS
function createCounter(start) {
  // `current` is captured by the returned closure and persists between calls.
  let current = start;
  return function () {
    return current++; // return current value, then advance
  };
}

// Demonstration

#### cons
t counter = createCounter(7);

#### cons
ole.log([counter(), counter(), counter()]); // [7, 8, 9]
Java
JAVA
import java.util.function.IntSupplier;
import java.util.ArrayList;
import java.util.List;

public class Solution {
    // Returns a supplier whose private state advances on each getAsInt().
    // A one-element array holds the mutable state captured by the lambda.
    public IntSupplier createCounter(int start) {
        int[] current = { start };
        return () -> current[0]++;   // return current, then advance
    }

    public static void main(String[] args) {

#### Solution
s = new Solution();
        IntSupplier counter = s.createCounter(7);
        List<Integer> out = new ArrayList<>();
        for (int i = 0; i < 3; i++) out.add(counter.getAsInt());
        System.out.println(out); // [7, 8, 9]
    }
}
C++
CPP
#include <functional>
#include <vector>
#include <iostream>

// The counting state lives inside a mutable lambda capture — a true closure.
// Each returned std::function has its own independent `current`.
std::function<int()> createCounter(int start) {
    int current = start;
    return [current] () mutable {
        return current++;   // return current, then advance
    };
}

int main() {
    auto counter = createCounter(7);
    std::vector<int> out;
    for (int i = 0; i < 3; i++) out.push_back(counter());
    for (int v : out) std::cout << v << " "; // 7 8 9
    std::cout << "\n";
    return 0;
}
C
C
#include <stdio.h>
#include <stdlib.h>

/* C has no closures, so we encapsulate the counting state in a struct.
   counterCreate seeds the state; counterNext returns the current value
   then advances it by one — semantically identical to a closure counter. */

typedef struct {
    int current;
} Counter;

Counter* counterCreate(int start) {
    Counter* c = (Counter*)malloc(sizeof(Counter));
    c->current = start;
    return c;
}

int counterNext(Counter* c) {
    return c->current++;   /* return current value, then increment */
}

void counterFree(Counter* c) {
    free(c);
}

/* Demonstration: produce the first `invocations` values into out[]. */
void generate(int start, int invocations, int* out) {
    Counter* c = counterCreate(start);
    for (int i = 0; i < invocations; i++) {
        out[i] = counterNext(c);
    }
    counterFree(c);
}

int main(void) {
    int out[3];
    generate(7, 3, out);
    for (int i = 0; i < 3; i++) printf("%d ", out[i]); /* 7 8 9 */
    printf("\n");
    return 0;
}

#### Follow-up
questions
Add a reset() operation that returns the generator to its original start.
Generalize the step so each call advances by an arbitrary amount (e.g. +2 or -1) instead of exactly +1.
Make the generator safe to call concurrently from multiple threads without missing or repeating values.
Support a peek() that reveals the next value without consuming it.

