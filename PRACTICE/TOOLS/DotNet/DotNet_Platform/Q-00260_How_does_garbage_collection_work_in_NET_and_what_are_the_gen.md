---
type: "practice"
difficulty: "Intermediate"
---

# How does garbage collection work in .NET and what are the generations (0, 1, 2)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Garbage Collection and Memory](<../../../../TOOLS/DotNet/DotNet_Platform/Garbage_Collection_and_Memory.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Garbage collection in .NET is a fundamental aspect of managing memory efficiently. It’s a process where the runtime automatically identifies and releases objects that are no longer being used by the application, freeing up memory. In .NET, garbage collection works through generations, which are primarily designed to optimize performance by segregating objects based on their lifespans.

The question is probing your understanding of how the .NET garbage collector operates under the hood, specifically focusing on the three generations: 0, 1, and 2. Let's break down each generation from first principles:

Generations in Garbage Collection

Understanding Generation 0

TL;DR: Generation 0 is where most short-lived objects are allocated.
In .NET, when an object is created, it starts its life in the generation with the shortest expected lifetime (Generation 0).

#### Example
When a method finishes executing and all local variables go out of scope, any objects that were only used locally are likely to be short-lived.

Understanding Generation 1

TL;DR: Objects that survive at least one garbage collection in Generation 0 move up to Generation 1.
After the initial GC, if an object is still being referenced and used by your application, it will be promoted to a longer generation (Generation 1).

#### Example
If you have a method that performs some processing on a collection of objects, and these objects are not immediately garbage collected because they're referenced elsewhere in the code, they might end up in Generation 1.

Understanding Generation 2

TL;DR: Objects that survive multiple collections in Generation 1 eventually move to Generation 2.
These are long-lived objects that your application uses continuously and frequently.

#### Example
If you have a global singleton or static object that is used throughout the lifecycle of an application, it will likely reside in Generation 2.
How It Works

When a garbage collection occurs:

Mark Phase:

The runtime starts with the roots (variables and references), marking all reachable objects.
Objects in Generation 0 are marked as they are first considered.

Sweep Phase:

Unmarked objects in Generation 0 are collected, freeing up memory.
Surviving objects move to Generation 1 or remain in Generation 0 if their size is very small.

Compact Phase (for Generation 1 and 2):

The runtime compacts the used space by moving surviving objects closer together, reducing fragmentation.

#### Common Mistakes
and Follow-Ups

Mistake: Assuming all objects are created in Generation 0 without exception.

#### Follow-Up
Objects can be created directly in Generation 1 or 2 if they are known to have a long lifespan (e.g., static objects).

Mistake: Ignoring the impact of object size on garbage collection.

#### Follow-Up
Smaller objects tend to stay in Generation 0, while larger objects may require more frequent promotions.

#### Closing Summary

Understanding how garbage collection works in .NET and the role of generations is crucial for writing efficient code. By managing memory effectively, you can improve application performance and avoid common pitfalls like excessive memory usage or performance degradation due to frequent garbage collections.

**Comparison:**

```
Generation	Characteristics	Examples
0	Short-lived objects	Local variables, temporary instances
1	Survivors from Gen 0	Objects referenced by Gen 0 survivors
2	Long-lived objects	Static members, global singletons
```

**Code:**

```csharp
// Example of a simple object creation in C#
public class MyClass
{
    public void CreateObject()
    {
        // Short-lived local variable
        var tempObj = new MyShortLivedClass();

        // Long-lived static reference
        staticReference = new MyLongLivedClass();
    }
}
```

**Example input / output:**

```text
- **Input**: Creating a `MyShortLivedClass` instance inside a method.
- **Output**: The object is allocated to Generation 0 and will likely be collected soon after the method exits.

- **Input**: Setting a static reference to an instance of `MyLongLivedClass`.
- **Output**: The object is promoted to Generation 2 and persists for the duration of the application.
```

**Data:**

```json
{
  "MyShortLivedClass": {},
  "MyLongLivedClass": {}
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Object Creation
6. Generation 0
7. Survive GC?
8. Swept
9. Move to Gen 1
10. Generation 1
11. Survive GC?
12. Swept
13. Move to Gen 2
14. Generation 2
15. Compact and Sweep

