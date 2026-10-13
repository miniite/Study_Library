---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a class and a struct in C#?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ LINQ and Collections](<../../../../TOOLS/DotNet/DotNet_Platform/LINQ_and_Collections.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with .NET, understanding the differences between classes and structs is crucial for effective object-oriented programming. Classes and structs are both reference types in C#, but they behave differently due to their underlying implementation.

TL;DR: The primary difference lies in how values are stored and passed around—classes use heap-based memory allocation (reference type), while structs use stack-based memory allocation (value type).

#### Triaging the Question

The question is asking for a fundamental explanation of classes versus structs in C#. Specifically, it's probing your understanding of:

The underlying data storage mechanisms.
When to use each based on performance and semantics.

#### First Principles Solution
1. Memory Allocation

Classes are reference types, meaning they always store a reference (pointer) to the actual object on the heap. This means that when you pass a class around or assign it to another variable, you're working with a pointer to the same memory location. Changes made through one reference will affect all references to that object.

Structs, on the other hand, are value types and store their entire data within the stack frame. When you pass a struct around or assign it to another variable, a copy of the struct's contents is created. Therefore, changes to one copy do not affect others.

2. Performance Considerations
Heap Allocation: Classes use heap allocation, which can be more expensive in terms of performance due to garbage collection and memory management overhead.
Stack Allocation: Structs use stack allocation, making them faster for small data structures because they don’t need to be managed by the garbage collector.
3. Size and Complexity
Size Limitations: C# structs can contain a maximum of 16 fields without being marked as packed (due to alignment issues). Classes have no such limitation.
Complexity: Use classes for more complex objects with many properties or methods, as they allow inheritance and provide better structure.

#### Closing Summary

In summary, the main differences between classes and structs in C# are related to memory allocation, performance, and use cases. Classes are used for larger, more complex objects where shared state might be an issue, while structs are ideal for small, simple data structures that should not share mutable state.

#### Common Mistakes/Follow-Ups
Not Considering Size: Failing to recognize the size limitations of structs can lead to performance issues.
Incorrect Inheritance Use: Using a class when a struct would be more appropriate in terms of performance and simplicity.
Overhead of Reference Types: Overusing reference types for small data structures without considering the memory implications.

**Code:**

```csharp
// Example of a simple class
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    // Methods, properties, etc.
}

// Example of a struct
public struct Point
{
    public int X;
    public int Y;

    // Constructor and other methods
}
```

