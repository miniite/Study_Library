---
type: "practice"
difficulty: "Intermediate"
---

# What is a local function and how does it differ from a lambda in terms of allocation and capture?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ LINQ and Collections](<../../../../TOOLS/DotNet/DotNet_Platform/LINQ_and_Collections.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Local functions and lambdas are both useful constructs in .NET that help with code organization and readability, but they have distinct differences when it comes to allocation and capture.

TL;DR: A local function is a named function defined inside another method or lambda expression. It can access variables from the outer scope directly without needing any explicit captures. In contrast, a lambda is an anonymous function that can capture values from its surrounding context but requires more overhead due to how it handles closures.

#### Triage
The question asks you to explain the differences between local functions and lambdas in terms of allocation and capture. You need to discuss the scope and behavior of variables within each construct to understand their implications.

Step-by-Step Explanation:

Local Functions:

A local function is defined inside another method or lambda expression.
It has a name and can be used just like any other named function.
Local functions have access to the outer scope by default, which means you don’t need to explicitly capture variables.
They are optimized for performance because they do not require capturing any state.

Lambdas:

A lambda is an anonymous function that can be used as a delegate or expression tree.
Lambdas have the ability to capture variables from their surrounding context, which means you need to explicitly mark which variables should be captured by value or reference.
Due to this capturing mechanism, lambdas are generally more expensive in terms of memory and performance compared to local functions.

#### First-Principles Solution

Allocation:

Local functions are allocated as regular methods on the stack when they are called.
Lambdas, being anonymous, need a separate delegate type for each use case, leading to more object allocations.

Capture Mechanism:

Local functions do not capture any state by default; if you want to access variables from the outer scope, you can explicitly declare them as ref or out.
Lambdas always need a closure mechanism because they are anonymous and must store references to their captured variables.

#### Performance Considerations

Local functions offer better performance due to their simpler structure and direct access to the outer scope.
Lambdas, while more flexible in terms of code organization, can introduce additional overhead for managing closures.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Always use local functions when you need a named function that accesses variables from its outer scope directly. This is both cleaner and faster.
Use lambdas for cases where you need an anonymous function and explicit capture of state is necessary, but be mindful of the performance implications.

#### Common mistakes
include overusing lambdas in situations where local functions would suffice, leading to unnecessary overhead. Conversely, failing to understand when a lambda’s capturing mechanism might be required can also lead to subtle bugs or performance issues.

**Code:**

```csharp
// Example of a Local Function
public void OuterMethod()
{
    int x = 10;

    // Local function that has access to x directly
    void LocalFunc() => Console.WriteLine(x);
    LocalFunc(); // Output: 10

    // Lambda with explicit capture (x by reference)
    Action localLambda = () => Console.WriteLine(x);
    localLambda(); // Output: 10
}

// Example of a Lambda
public void OuterMethod()
{
    int x = 10;

    // Lambda that captures x by value
    Func<int> lambda = () => x;
    x = 20; // Changing the outer variable

    Console.WriteLine(lambda()); // Output: 10, not 20 because of value capture
}
```

