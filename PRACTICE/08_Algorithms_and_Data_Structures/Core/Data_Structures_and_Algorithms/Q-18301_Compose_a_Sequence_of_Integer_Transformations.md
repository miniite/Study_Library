---
type: "practice"
difficulty: "Other"
---

# Compose a Sequence of Integer Transformations

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Higher-Order Functions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Higher-Order_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list of transformations t = [t1, t2, ..., tk]. Each transformation is a function that takes a single integer and returns a single integer.

Build and return a single new function combined that is the composition of all the transformations applied right to left. Formally:

CODE
combined(x) = t1( t2( ... tk(x) ... ) )

The rightmost transformation is applied first to the input, its result is fed into the next transformation to its left, and so on until the leftmost transformation produces the final answer.

If the list of transformations is empty, combined must be the identity function, i.e. combined(x) = x.

You may assume every transformation accepts exactly one integer and returns one integer. For the purpose of checking your answer, we evaluate the returned function at a given integer x and report combined(x).

Input

A list of k unary integer transformations (possibly empty).
An integer x at which to evaluate the composed function.

Output

The value combined(x) produced by applying the transformations right to left.

#### Cons
traints

-1000 <= x <= 1000
0 <= k <= 1000
Each transformation maps one integer to one integer.
For the evaluated cases, every intermediate result fits in a signed 64-bit integer.

#### Example
s & test cases
TEXT
Input: transformations = [x -> x - 3, x -> x * x, x -> x + 5], x = 2 -> Output: 46
Why: right to left: (2+5)=7, then 7*7=49, then 49-3=46 — the standard right-to-left flow.

Input: transformations = [x -> 3 * x, x -> 3 * x], x = 2 -> Output: 18
Why: innermost first 3*2=6, then 3*6=18 — repeated transformations chain correctly.

Input: transformations = [], x = 7 -> Output: 7
Why: empty list must behave as the identity function, returning the input unchanged.

Input: transformations = [x -> x * x], x = -4 -> Output: 16
Why: single-element list applies that one transformation; negative input squares to positive.

Input: transformations = [x -> x + 1, x -> 2 * x], x = -1000 -> Output: -1999
Why: edge value; 2*(-1000)=-2000 then -2000+1=-1999 confirms right-to-left order and negatives.

#### Approach

This is a right fold (reduce-from-the-right) over the list of functions. Start with the input value as an accumulator, then apply the transformations in reverse index order (last element first), threading the running result through each one. Returning a closure that captures the list gives a reusable composed function; the empty-list case naturally returns the input unchanged, which is exactly the identity function. The core pattern is higher-order functions / function composition via folding.

Complexity

Time: O(k) per evaluation, where k is the number of transformations — each is applied exactly once. Space: O(1) extra beyond storing the transformation list / returned closure, since evaluation uses a single running accumulator.

Python
PYTHON
from typing import Callable, List

def compose(transformations: List[Callable[[int], int]]) -> Callable[[int], int]:
    """Return a function equal to the right-to-left composition of the inputs.
    An empty list yields the identity function."""
    def combined(x: int) -> int:
        result = x
        for t in reversed(transformations):
            result = t(result)
        return result
    return combined
JavaScript
JAVASCRIPT
/**
 * @param {Function[]} transformations
 * @return {Function} the right-to-left composition; identity when empty
 */
function compose(transformations) {
  return function (x) {
    let result = x;
    for (let i = transformations.length - 1; i >= 0; i--) {
      result = transformations[i] (result);
    }
    return result;
  };
}
Java
JAVA
import java.util.List;
import java.util.function.LongUnaryOperator;

public class Solution {
    // Returns a function equal to the right-to-left composition of the inputs.
    // An empty list yields the identity function.
    public static LongUnaryOperator compose(List<LongUnaryOperator> transformations) {
        return x -> {
            long result = x;
            for (int i = transformations.size() - 1; i >= 0; i--) {
                result = transformations.get(i).applyAsLong(result);
            }
            return result;
        };
    }
}
C++
CPP
#include <functional>
#include <vector>

// Returns a closure equal to the right-to-left composition of the inputs.
// An empty vector yields the identity function.
std::function<long long(long long)>
compose(const std::vector<std::function<long long(long long)>>& transformations) {
    return [transformations] (long long x) -> long long {
        long long result = x;
        for (int i = static_cast<int>(transformations.size()) - 1; i >= 0; --i) {
            result = transformations[i] (result);
        }
        return result;
    };
}
C
C
#include <stddef.h>

/* C has no closures, so we expose the evaluated composition directly:
   compose_eval applies t[0] (t[1] (...t[n-1] (x)...)) — i.e. right to left.
   With n == 0 this is the identity and returns x unchanged. */

typedef long long (*IntFn)(long long);

long long compose_eval(IntFn *transformations, size_t n, long long x) {
    long long result = x;
    for (size_t i = n; i-- > 0; ) {          /* walk indices n-1 .. 0 */
        result = transformations[i] (result);
    }
    return result;
}

#### Follow-up
questions
Compose the functions left to right instead of right to left; how does the loop change?
Implement it without an explicit loop, using a single reduce/fold expression.
Generalize to transformations that take and return multiple values (variadic composition).
Make the composition lazy so a transformation is only invoked when its result is actually needed.

