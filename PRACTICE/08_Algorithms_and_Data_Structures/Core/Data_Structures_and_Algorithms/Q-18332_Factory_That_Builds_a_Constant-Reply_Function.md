---
type: "practice"
difficulty: "Other"
---

# Factory That Builds a Constant-Reply Function

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Closures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Closures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Implement a zero-argument factory function makeConstantReply(). Calling the factory must return a new function reply. The returned function reply must always produce the exact string "Ready to Launch", no matter what arguments (if any) are passed to it.

Key points:

makeConstantReply itself takes no arguments and its only job is to return another function.
The returned reply function completely ignores whatever arguments it is called with — zero arguments, one, or many, of any type — and unconditionally returns the fixed string "Ready to Launch".
Each call to the factory yields a function with the same constant behavior.

Input

args: the (ignored) list of arguments passed to the returned reply function on a single invocation.

Output

The string "Ready to Launch" on every invocation, regardless of args.

#### Cons
traints

0 <= args.length <= 10
Arguments may be of any type and must not affect the output.
The factory must return a fresh callable; it must not print or perform I/O.

#### Example
s & test cases
TEXT
Input: args = [] -> Output: "Ready to Launch"
Why: With no arguments the returned function still yields the fixed string.

Input: args = [{}, null, 42] -> Output: "Ready to Launch"
Why: A mix of object, null, and number is passed but every argument is ignored.

Input: args = ["anything", "at", "all"] -> Output: "Ready to Launch"
Why: String arguments are ignored; the constant output is unchanged.

Input: args = [true, [1, 2, 3], -7.5] -> Output: "Ready to Launch"
Why: Booleans, arrays, and floats are all ignored; output is invariant.

Input: two separate factory calls, each invoked once with [] -> Output: "Ready to Launch" and "Ready to Launch"
Why: Every function produced by the factory shares the same constant behavior.

#### Approach

This is the simplest higher-order-function / closure pattern: a factory whose sole purpose is to return another function. The inner function is a constant function — it discards every argument via variadic capture (...args / *args, **kwargs) and always evaluates to the same literal. The factory returns this inner function as a first-class value, demonstrating that functions can be produced and passed around like data. In statically typed languages the "returns a function" idea is expressed with a functional type (std::function / Supplier / a function pointer); since the inner function ignores its inputs, it is modeled cleanly as a nullary callable.

Complexity

Time: O(1) — the factory and the returned function each do constant work. Space: O(1) — no state is captured or allocated beyond the returned callable. Justification: producing and invoking a constant-returning closure involves no loops, no data structures, and no dependence on input size.

Python
PYTHON
def make_constant_reply():
    """Return a function that ignores all arguments and always
    returns the fixed string 'Ready to Launch'."""
    def reply(*args, **kwargs):
        return "Ready to Launch"
    return reply
JavaScript
JAVASCRIPT
/**
 * @return {Function} a function that always returns "Ready to Launch"
 */
function makeConstantReply() {
  return function (...args) {   // args accepted but ignored
    return "Ready to Launch";
  };
}
Java
JAVA
import java.util.function.Supplier;

class Solution {
    // makeConstantReply() returns a Supplier that always produces the fixed string.
    public static Supplier<String> makeConstantReply() {
        return () -> "Ready to Launch";
    }
}
C++
CPP
#include <functional>
#include <string>
using namespace std;

// makeConstantReply() returns a callable that always yields the fixed string.
// The returned lambda takes no arguments because it ignores all inputs anyway.
function<string()> makeConstantReply() {
    return [] () -> string {
        return "Ready to Launch";
    };
}
C
C
/* The returned "function" is a function pointer. The inner function ignores
   any inputs and returns the fixed string; the factory hands back its address. */

typedef const char *(*ReplyFn)(void);

static const char *constant_reply(void) {
    return "Ready to Launch";   /* ignores anything a caller might pass */
}

/* makeConstantReply() -> returns the constant-reply function */
ReplyFn make_constant_reply(void) {
    return constant_reply;
}

#### Follow-up
questions
Parameterize the factory so the constant string is supplied when the factory is called and captured by the returned function.
Make the returned function echo how many times it has been invoked while still returning the constant (add captured state).
Explain why the returned function keeps working after the factory has returned (closure over its environment / lifetime of captured state).
Return a function that lazily computes an expensive constant once and caches it for later calls.

