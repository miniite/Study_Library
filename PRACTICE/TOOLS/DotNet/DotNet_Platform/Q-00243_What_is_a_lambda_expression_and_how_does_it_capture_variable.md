---
type: "practice"
difficulty: "Intermediate"
---

# What is a lambda expression and how does it capture variables (closures)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ LINQ and Collections](<../../../../TOOLS/DotNet/DotNet_Platform/LINQ_and_Collections.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A lambda expression in .NET is a convenient way to create anonymous functions, which can be used where a method needs to be passed as an argument or stored as a variable. What makes lambda expressions particularly powerful is their ability to capture variables from the surrounding context, creating what's known as closures. This allows for more flexible and concise code.

When the interviewer asks about lambda expressions in .NET and how they capture variables (closures), they are probing your understanding of functional programming concepts, method syntax, variable scoping rules, and practical use cases. Assumptions include familiarity with C# or VB.NET since these languages support lambda expressions. The key points to cover are the definition, syntax, usage, and implications of closures.

Let's break this down step by step:

Definition: A lambda expression is a shorthand way to define small functions without giving them a name.
Syntax: In C#, it looks like expression => statement or (parameters) => expression.
Capturing Variables (Closures): When a lambda captures variables from its surrounding context, they are stored as part of the function's state. This allows the lambda to access and modify those variables even after the original scope has exited.

#### First-Principles Solution

#### Example
Basic Lambda Expression

#### Cons
ider this simple example in C#:

CSHARP
int number = 10;
Func<int> getNumber = () => number;

number = 20; // Change the value of `number`

#### Cons
ole.WriteLine(getNumber()); // Outputs "10" because the lambda captured the initial value
Explanation:
The getNumber lambda captures the variable number.
Even though we change the value of number, the lambda still returns the original value it captured.

#### Example
Lambda with Multiple Captured Variables
CSHARP
int a = 1, b = 2;
Func<int> addNumbers = () => a + b;

a = 3; // Change the value of `a`

#### Cons
ole.WriteLine(addNumbers()); // Outputs "5" because the lambda captured both values when defined
Explanation:
The lambda captures both a and b at the time it was defined.
Modifying these variables afterward doesn't affect the lambda's behavior.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, lambda expressions in .NET are a powerful feature that allow for concise function definitions. They can capture variables from their surrounding context (creating closures), which is both useful and potentially dangerous if not managed carefully. For instance, mutable captured variables can lead to bugs if they change unexpectedly.

#### Common mistakes
include:

Not understanding the scoping rules around variable captures.
Misusing lambdas in scenarios where capturing state might cause issues due to changes outside the lambda's scope.

#### Follow-ups
could involve discussing when and why closures are beneficial or harmful, as well as exploring related concepts like delegates and asynchronous programming patterns that leverage lambda expressions.

