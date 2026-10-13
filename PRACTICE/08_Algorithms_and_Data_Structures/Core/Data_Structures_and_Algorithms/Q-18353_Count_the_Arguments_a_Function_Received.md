---
type: "practice"
difficulty: "Other"
---

# Count the Arguments a Function Received

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Variadic Functions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Variadic_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Build a variadic function argumentCount that can be invoked with any number of arguments — including none at all — and returns exactly how many arguments it was given on that call. The values themselves are irrelevant; only the count of arguments matters.

For example, invoking it with a single value must return 1, invoking it with three values (of any types, in any mix) must return 3, and invoking it with no arguments must return 0.

Input / Output. Conceptually you are given the list of values that were handed to the function on one invocation. Return the number of values in that list. Passing zero values returns 0.

#### Cons
traints.

0 <= number_of_arguments <= 100
Each argument may be a value of any type (number, text, object, null, etc.); its type and contents do not affect the result.
The function must report the count of arguments actually supplied at the call site, not the number of named parameters it declares.

#### Example
s & test cases
TEXT
Input: args = [42] -> Output: 1
Why: exactly one value was supplied, so the count is 1.

Input: args = [obj, null, "x"] -> Output: 3
Why: three values of mixed type were supplied; type and content are ignored, only the count matters.

Input: args = [] -> Output: 0
Why: edge case — no arguments were supplied, so the count is 0 (minimum bound).

Input: args = [true, true] -> Output: 2
Why: duplicate/identical values still each count once, giving 2.

Input: args = [1, 2, 3, ... , 100] (100 values) -> Output: 100
Why: maximum bound — every supplied argument is counted.

#### Approach

The core idea is variadic (rest) parameters: instead of declaring a fixed number of named parameters, the function accepts an arbitrary argument list. The language runtime packs those arguments into a single sequence — a rest array in JavaScript, *args tuple in Python, an Object... array in Java — whose length is the answer. In C and C++, which have no runtime argument-length introspection, the count is resolved statically: C++ uses a variadic template and sizeof..., while C uses a variadic preprocessor macro that expands to the number of tokens. All five report the exact number of arguments supplied at the call site in constant time.

Complexity

Time: O(1) — the count is available immediately, either from the length of the packed argument sequence (JS/Python/Java) or computed by the compiler (C/C++). Space: O(1) auxiliary — no data structure proportional to input is built to obtain the count (the argument container itself is inherent to the call, not extra work).

Python
PYTHON
def argument_count(*args):
    """Return how many positional arguments were supplied on this call."""
    return len(args)          # *args packs every argument into a tuple

if __name__ == "__main__":
    print(argument_count())               # 0
    print(argument_count(42))             # 1
    print(argument_count({}, None, "x"))  # 3
    print(argument_count(True, True))     # 2
JavaScript
JS
function argumentCount(...args) {
  // The rest parameter gathers every argument into an array; its length
  // is exactly the number of arguments the caller supplied.
  return args.length;
}

// Demonstration

#### cons
ole.log(argumentCount());            // 0

#### cons
ole.log(argumentCount(42));          // 1

#### cons
ole.log(argumentCount({}, null, "x")); // 3

#### cons
ole.log(argumentCount(true, true));  // 2
Java
JAVA
public class Solution {
    // Varargs collects every supplied argument into an Object[] whose length
    // is exactly the number of arguments passed at the call site.
    public static int argumentCount(Object... args) {
        return args.length;
    }

    public static void main(String[] a) {
        System.out.println(argumentCount());                        // 0
        System.out.println(argumentCount(42));                      // 1
        System.out.println(argumentCount(new Object(), null, "x")); // 3
        System.out.println(argumentCount(true, true));              // 2
    }
}
C++
CPP
#include <cstddef>
#include <iostream>

// A variadic template packs every argument into the parameter pack Args.
// sizeof...(Args) is the number of arguments, known at compile time in O(1).
template <typename... Args>

#### cons
texpr std::size_t argumentCount(Args&&...) {
    return sizeof...(Args);
}

int main() {
    std::cout << argumentCount() << "\n";              // 0
    std::cout << argumentCount(42) << "\n";            // 1
    std::cout << argumentCount(0, nullptr, "x") << "\n"; // 3
    std::cout << argumentCount(true, true) << "\n";    // 2
    return 0;
}
C
C
#include <stdio.h>

/* A C function cannot introspect how many arguments it received, so the count is
   resolved at compile time by a variadic macro. With the GNU ", ##__VA_ARGS__"
   extension the leading placeholder comma is dropped when the list is empty, so
   ARG_COUNT() correctly yields 0. Supports 0..10 arguments here and extends by
   adding more slots to the two lists below. */

#define ARG_COUNT(...) \
    ARG_COUNT_IMPL(0, ##__VA_ARGS__, 10,9,8,7,6,5,4,3,2,1,0)
#define ARG_COUNT_IMPL(_0,_1,_2,_3,_4,_5,_6,_7,_8,_9,_10, N, ...) N

int main(void) {
    printf("%d\n", ARG_COUNT());              /* 0 */
    printf("%d\n", ARG_COUNT(42));            /* 1 */
    printf("%d\n", ARG_COUNT(0, 0, "x"));     /* 3 */
    printf("%d\n", ARG_COUNT(1, 1));          /* 2 */
    return 0;
}

#### Follow-up
questions
Return the collected argument list itself, not just its size.
Count only the numeric arguments (or only those of a given type) within a mixed list.
Distinguish an explicitly passed null/undefined from an argument that was omitted entirely.
Enforce an upper limit and reject a call that supplies more arguments than allowed.

