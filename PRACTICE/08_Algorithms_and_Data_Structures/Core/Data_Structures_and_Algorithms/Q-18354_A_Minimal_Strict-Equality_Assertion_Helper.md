---
type: "practice"
difficulty: "Other"
---

# A Minimal Strict-Equality Assertion Helper

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Closures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Closures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Build a small testing helper. Write a factory check(value) that captures a value and returns an object exposing exactly two operations:

equals(other) — returns true if value and other are strictly equal (same type and same value, with no implicit type conversion). If they are not strictly equal, it must raise an error whose message is "Not Equal".
differs(other) — returns true if value and other are strictly unequal. If they are strictly equal, it must raise an error whose message is "Equal".

"Strict" means no coercion: two values are treated as equal only when they share the same type and compare equal. Exactly one outcome happens on each call — either the method returns true, or it raises the corresponding error.

Input / Output. Conceptually each operation is check(a).equals(b) or check(a).differs(b). The result is either the boolean true, or the raised error message ("Not Equal" for a failed equals, "Equal" for a failed differs).

#### Cons
traints.

The captured value and the compared value may be of any type.
Comparison is strict — no implicit conversion between types is performed.
Each call resolves to exactly one of: return true, or raise the stated error.

#### Example
s & test cases
TEXT
Input: check(5).equals(5) -> Output: true
Why: same type and same value, so equals succeeds.

Input: check(5).equals(9) -> Output: error "Not Equal"
Why: values differ, so equals raises the "Not Equal" error.

Input: check(5).differs(9) -> Output: true
Why: the two values are strictly unequal, so differs succeeds.

Input: check(5).differs(5) -> Output: error "Equal"
Why: the two values are strictly equal, so differs raises the "Equal" error.

Input: check("a").equals("a") -> Output: true
Why: strings of the same type and value are strictly equal, so equals succeeds.

#### Approach

The pattern is a closure/factory that captures state and returns behavior. check binds the given value and hands back an object with two methods, equals and differs, that compare a later value against the captured one using strict equality (same type, no coercion). Success returns true; failure is signalled the idiomatic way per language — throwing an exception in JavaScript, C++, Java, and Python, and returning a status with an error-message out-parameter in C, which lacks exceptions. Static typing (C's long, the template T in C++, the generic T in Java) or an explicit type(...) is type(...) check in Python enforces the strict "same type" requirement.

Complexity

Time: O(1) per operation — a single strict comparison of two values. Space: O(1) — the helper stores just the one captured value; no growth with input.

Python
PYTHON
class Checker:
    def __init__(self, value):
        self._value = value

    def _strict_equal(self, other):
        # Strict = same type AND equal value; no implicit coercion.
        return type(self._value) is type(other) and self._value == other

    def equals(self, other):
        if self._strict_equal(other):
            return True
        raise ValueError("Not Equal")

    def differs(self, other):
        if not self._strict_equal(other):
            return True
        raise ValueError("Equal")

def check(value):
    return Checker(value)

if __name__ == "__main__":
    print(check(5).equals(5))    # True
    print(check(5).differs(9))   # True
    try:
        check(5).equals(9)
    except ValueError as e:
        print(e)                 # Not Equal
    try:
        check(5).differs(5)
    except ValueError as e:
        print(e)                 # Equal
JavaScript
JS
function check(value) {
  // `value` is captured by the returned object's methods (a closure). Strict
  // equality (===) performs no type coercion; failure throws the required message.
  return {
    equals(other) {
      if (value === other) return true;
      throw new Error("Not Equal");
    },
    differs(other) {
      if (value !== other) return true;
      throw new Error("Equal");
    },
  };
}

// Demonstration

#### cons
ole.log(check(5).equals(5));   // true

#### cons
ole.log(check(5).differs(9));  // true
try { check(5).equals(9); } catch (e) { console.log(e.message); }  // Not Equal
try { check(5).differs(5); } catch (e) { console.log(e.message); } // Equal
Java
JAVA
import java.util.Objects;

public class Solution {
    // The generic type T enforces the "same type" half of strict equality, while
    // Objects.equals handles null-safe value comparison. Failure throws, per spec.
    static class Checker<T> {
        private final T value;
        Checker(T value) { this.value = value; }

        boolean equals(T other) {
            if (Objects.equals(value, other)) return true;
            throw new RuntimeException("Not Equal");
        }
        boolean differs(T other) {
            if (!Objects.equals(value, other)) return true;
            throw new RuntimeException("Equal");
        }
    }

    static <T> Checker<T> check(T value) { return new Checker<>(value); }

    public static void main(String[] a) {
        System.out.println(check(5).equals(5));   // true
        System.out.println(check(5).differs(9));  // true
        try { check(5).equals(9); }
        catch (RuntimeException e) { System.out.println(e.getMessage()); } // Not Equal
        try { check(5).differs(5); }
        catch (RuntimeException e) { System.out.println(e.getMessage()); } // Equal
    }
}
C++
CPP
#include <stdexcept>
#include <string>
#include <iostream>

// The captured value lives inside the Checker; the template parameter T makes the
// comparison strict — equals/differs only accept another T, so no cross-type
// coercion can occur. Failure is signalled by throwing, mirroring the spec.
template <typename T>
class Checker {
    T value;
public:
    explicit Checker(T v) : value(v) {}
    bool equals(const T& other) const {
        if (value == other) return true;
        throw std::runtime_error("Not Equal");
    }
    bool differs(const T& other) const {
        if (!(value == other)) return true;
        throw std::runtime_error("Equal");
    }
};

template <typename T>
Checker<T> check(T value) { return Checker<T>(value); }

int main() {
    std::cout << std::boolalpha << check(5).equals(5) << "\n";   // true
    std::cout << check(5).differs(9) << "\n";                    // true
    try { check(5).equals(9); }
    catch (const std::exception& e) { std::cout << e.what() << "\n"; } // Not Equal
    try { check(5).differs(5); }
    catch (const std::exception& e) { std::cout << e.what() << "\n"; } // Equal
    return 0;
}
C
C
#include <stdio.h>

/* C has no exceptions or method objects, so we capture the value in a struct and
   expose equals/differs that return 1 on success. On failure they write the error
   text ("Not Equal" / "Equal") into *err and return 0 — the C analog of throwing.
   Static typing already guarantees both operands share a type, which models the
   "strict" (no-coercion) part of the comparison. */

typedef struct { long value; } Checker;

Checker check(long value) {
    Checker c;
    c.value = value;
    return c;
}

int equals(Checker c, long other, const char** err) {
    if (c.value == other) return 1;
    *err = "Not Equal";
    return 0;
}

int differs(Checker c, long other, const char** err) {
    if (c.value != other) return 1;
    *err = "Equal";
    return 0;
}

int main(void) {

#### cons
t char* err = NULL;
    printf("%d\n", equals(check(5), 5, &err));                 /* 1 */
    err = NULL;
    if (!equals(check(5), 9, &err)) printf("%s\n", err);       /* Not Equal */
    err = NULL;
    printf("%d\n", differs(check(5), 9, &err));                /* 1 */
    err = NULL;
    if (!differs(check(5), 5, &err)) printf("%s\n", err);      /* Equal */
    return 0;
}

#### Follow-up
questions
Support chaining a not prefix so that check(x).not.equals(y) inverts the assertion.
Include the two compared values inside the raised error message for clearer diagnostics.
Extend to deep/structural equality so arrays and objects compare by contents, not identity.
Add ordered assertions such as greaterThan / lessThan alongside equals/differs.

