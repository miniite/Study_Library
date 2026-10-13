---
type: "practice"
difficulty: "Other"
---

# Invoke a Function With an Explicit Receiver Object

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ JavaScript](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/JavaScript.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In JavaScript, every function call carries a hidden receiver known as this. When a plain function is invoked directly, this is not bound to any specific object, so property reads such as this.label evaluate to undefined. Sometimes you want to run a function as if it were a method of a chosen object for a single call, so that this inside the function refers to that object.

Add a new method named runWithContext to every function. The method takes an object context as its first argument, followed by any number of extra arguments. Calling someFunction.runWithContext(context, a, b, ...) must:

Execute someFunction with this bound to context.
Forward the extra arguments a, b, ... to someFunction in order.
Return whatever someFunction returns.

You must implement this without using the language's built-in Function.prototype.call. (Building the behavior from other primitives — a temporary property, a symbol key, or apply — is fine.)

For example, given:

CODE
function priceTag(base, rate) {
  return `Charge for ${this.label} is ${base * rate}`;
}

Calling priceTag(8, 1.25) produces "Charge for undefined is 10" because this is unset. But priceTag.runWithContext({ label: "kettle" }, 8, 1.25) must produce "Charge for kettle is 10", because this was bound to { label: "kettle" } and 8, 1.25 were forwarded.

Input
fn: the function to invoke.
context: a non-null object that becomes this for this one call.
Zero or more additional arguments to forward to fn.
Output
The value returned by fn when run with this === context and the forwarded arguments.

#### Cons
traints
context is an object and is never null.
Total number of arguments passed (context plus forwarded ones) is between 1 and 100.
2 <= JSON.stringify(context).length <= 10^5.
runWithContext must not rely on the built-in Function.prototype.call.

#### Example
s & test cases
TEXT
Input: fn = function(y){ return this.x + y; }, context = {x: 5}, args = [7] -> Output: 12
Why: `this.x` resolves to 5 and 7 is forwarded, so 5 + 7 = 12. Basic single-argument case.

Input: fn = function(base, rate){ return `Charge for ${this.label} is ${base * rate}`; }, context = {label: "kettle"}, args = [8, 1.25] -> Output: "Charge for kettle is 10"
Why: Two arguments forwarded and `this.label` bound to "kettle"; 8 * 1.25 = 10.

Input: fn = function(){ return this.name; }, context = {name: "core"}, args = [] -> Output: "core"
Why: No forwarded arguments; only the receiver matters and `this.name` is read.

Input: fn = function(a, b, c){ return this.k * (a + b + c); }, context = {k: 2}, args = [1, 2, 3] -> Output: 12
Why: Multiple forwarded arguments summed (6) and scaled by `this.k` (2) = 12.

Input: fn = function(){ return Object.keys(this).length; }, context = {a:1, b:2, c:3}, args = [] -> Output: 3
Why: Confirms `this` truly points at the given object; it has 3 keys. Edge case reading the context's own structure.

#### Approach

The core idea is receiver binding: run a function so that a chosen object acts as its this/receiver, while forwarding the remaining arguments unchanged. In JavaScript the cleanest polyfill for Function.prototype.call exploits the language rule that a function invoked as obj.method(...) gets obj as this. So temporarily store the function on the context under a unique Symbol key (unique to avoid clobbering real properties), call it as a method with the spread arguments, then delete the temporary key so the caller's object is left untouched. This avoids the built-in call entirely. Across languages that have no rebindable implicit receiver (C, C++, Java, Python), the equivalent technique is to pass the context explicitly as the first ('self'/receiver) parameter and forward the rest — Python's types.MethodType binds the receiver literally, giving the closest analog to setting this. The pattern is function binding / higher-order dispatch built from primitives.

Complexity

Time: O(k) where k is the number of forwarded arguments — the work is spreading/forwarding the argument list; the invocation itself is O(1) overhead plus the cost of running the user's function. Space: O(k) for the forwarded argument list (plus O(1) for the temporary symbol property in the JavaScript version). Attaching and deleting the temporary key is constant time and leaves the context object unchanged after the call.

Python
PYTHON
import types

def invoke_with(fn, context, *args):
    """Bind `context` as the receiver of `fn` for one call, then forward args.

    types.MethodType attaches `context` as the implicit first ('self') argument,
    mirroring how JavaScript sets `this` for a single invocation. We do not use
    any built-in call/apply helper other than binding.
    """
    bound = types.MethodType(fn, context)
    return bound(*args)

class Bag:
    def __init__(self, **kw):
        self.__dict__.update(kw)

if __name__ == "__main__":
    def add(self, y):
        return self.x + y

    print(invoke_with(add, Bag(x=5), 7))  # 12

    def price_tag(self, base, rate):
        return f"Charge for {self.label} is {int(base * rate)}"

    print(invoke_with(price_tag, Bag(label="kettle"), 8, 1.25))  # Charge for kettle is 10
JavaScript
JAVASCRIPT
// Canonical solution: add runWithContext to every function via the prototype.
// The trick: temporarily attach the function to `context` under a unique symbol
// key, then invoke it as a method so `this` resolves to `context`. No use of the
// built-in Function.prototype.call.
Function.prototype.runWithContext = function (context, ...args) {

#### cons
t key = Symbol("fn");
  context[key] = this;          // attach; invoking as a method binds `this`

#### cons
t result = context[key] (...args);
  delete context[key];          // clean up so we never mutate the caller's object
  return result;
};

// Examples:
// const add = function (y) { return this.x + y; };
// add.runWithContext({ x: 5 }, 7); // 12
//
// const priceTag = function (base, rate) {
//   return `Charge for ${this.label} is ${base * rate}`;
// };
// priceTag.runWithContext({ label: "kettle" }, 8, 1.25); // "Charge for kettle is 10"
Java
JAVA
import java.util.*;

public class Solution {
    // A contextual function: takes its receiver, then the forwarded arguments.
    interface ContextFn<C, R> {
        R apply(C context, Object... args);
    }

    // invokeWith binds `context` as the receiver and forwards the rest,
    // without any built-in call/apply reflection trick.
    static <C, R> R invokeWith(ContextFn<C, R> fn, C context, Object... args) {
        return fn.apply(context, args);
    }

    public static void main(String[] args) {
        Map<String, Object> context = new HashMap<>();
        context.put("label", "kettle");

        ContextFn<Map<String, Object>, String> priceTag = (self, a) -> {
            double base = (double) a[0], rate = (double) a[1];
            return "Charge for " + self.get("label") + " is " + (int) (base * rate);
        };

        System.out.println(invokeWith(priceTag, context, 8.0, 1.25)); // Charge for kettle is 10
    }
}
C++
CPP
#include <bits/stdc++.h>
using namespace std;

// C++ has no rebindable `this` for free functions; the analog is to pass the
// receiver explicitly and forward the remaining arguments without any built-in
// call/apply — a plain templated dispatcher does exactly that.
template <typename Context, typename Fn, typename... Args>
auto invokeWith(Fn&& fn, Context& context, Args&&... args)
    -> decltype(fn(context, std::forward<Args>(args)...)) {
    return fn(context, std::forward<Args>(args)...);
}

int main() {
    struct Ctx { string label; } c{"kettle"};

    auto priceTag = [] (Ctx& self, double base, double rate) {
        return "Charge for " + self.label + " is " + to_string((int)(base * rate));
    };

    cout << invokeWith(priceTag, c, 8.0, 1.25) << "\n"; // Charge for kettle is 10
}
C
C
#include <stdio.h>

/* C has no implicit receiver; the faithful analog is to pass the context
   (receiver) explicitly as the first parameter and forward the rest. */

typedef struct { const char *label; } Context;

/* A "contextual function": receives its receiver first, then real arguments. */
static double priceTag(const Context *self, double base, double rate) {
    printf("Charge for %s is ", self->label);
    return base * rate;
}

/* invokeWith binds `context` as the receiver, then forwards the argument. */
static double invokeWith(double (*fn)(const Context *, double, double),

#### cons
t Context *context, double base, double rate) {
    return fn(context, base, rate);
}

int main(void) {
    Context c = { "kettle" };
    double out = invokeWith(priceTag, &c, 8.0, 1.25);
    printf("%g\n", out); /* Charge for kettle is 10 */
    return 0;
}

#### Follow-up
questions
Implement applyPolyfill(context, argsArray) that takes the forwarded arguments as a single array instead of a variadic list.
Implement bindPolyfill(context, ...preset) that returns a new function permanently bound to context and to any preset leading arguments.
What should happen when context is null or undefined — how does the real call treat it in strict vs non-strict mode, and how would you emulate that?
Why use a Symbol key rather than a string property name, and how does that prevent collisions with existing keys on context?

