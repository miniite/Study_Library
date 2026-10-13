---
type: "practice"
difficulty: "Basic"
---

# Write a memoized version of a recursive Fibonacci function and explain the speedup.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tail Latency with Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Tail_Latency_with_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we'll start by writing a basic recursive implementation of the Fibonacci sequence, then enhance it with caching to reduce redundant computations. The goal is to understand how memoization can significantly improve the performance of our function.

#### Triaging the Question

The interviewer is likely probing your understanding of recursion and caching mechanisms. They want you to demonstrate that you can apply memoization to optimize a recursive algorithm, thereby reducing the time complexity from exponential (O(2^n)) to linear (O(n)). Key assumptions include:

You have basic knowledge of Python or another language.
The Fibonacci function will be implemented recursively.

#### First Principles Solution
Step 1: Basic Recursive Implementation

First, let's write a simple recursive implementation of the Fibonacci function. This version has an exponential time complexity due to redundant calculations.

PYTHON
def fibonacci(n):
    if n <= 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fibonacci(n - 1) + fibonacci(n - 2)
Step 2: Memoization with a Dictionary

Memoization involves storing the results of expensive function calls and reusing them when the same inputs occur again. We can use a dictionary to cache the Fibonacci values.

PYTHON
def memoized_fibonacci(n, cache={}):
    if n in cache:
        return cache[n]
    elif n <= 1:
        result = n
    else:
        result = memoized_fibonacci(n - 1) + memoized_fibonacci(n - 2)
    cache[n] = result
    return result
Step 3: Dry-Run and Explanation

Let's dry-run the function for fibonacci(5):

PLAINTEXT
memoized_fibonacci(5) -> (Not cached, calculate)
             /                 \
memoized_fibonacci(4) + memoized_fibonacci(3) -> (Not cached, calculate each)
               ... (similar recursive calls until base cases are reached)

Each call to memoized_fibonacci checks if the result is already in the cache. If it's not, it calculates the value and stores it before returning.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we transformed a basic recursive Fibonacci function into a memoized version by using a dictionary for caching results. This approach ensures that each unique call to fibonacci(n) is computed only once, significantly improving performance.

#### Common mistakes
include:

Not using the cache properly (e.g., forgetting to check if the result is already cached).
Failing to handle edge cases like negative numbers or zero.
Overlooking the space complexity introduced by caching.

#### Follow-ups
might involve discussing additional optimizations, such as using a more efficient data structure for the cache or comparing this approach with other optimization techniques like iterative solutions.

