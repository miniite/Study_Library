---
type: "practice"
difficulty: "Basic"
---

# What is the difference between an array and a List<T> in .NET?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ LINQ and Collections](<../../../../TOOLS/DotNet/DotNet_Platform/LINQ_and_Collections.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with collections in .NET, both arrays and List<T> are commonly used to store elements. However, there are several key differences between them that affect their suitability for different scenarios.

#### Triage

The core of this question is to explain the fundamental differences between an array and a List<T> in .NET. The interviewer might be probing your understanding of when to use each structure based on performance, memory usage, mutability, and ease of use. You need to make some assumptions about the context, such as whether we are discussing C# or another language, and the typical operations performed.

#### First-Principles Solution
Arrays in .NET

Arrays in .NET are fixed-size structures that store a collection of elements of the same data type. They have a set capacity which is defined at creation time. When an array reaches its maximum size, it cannot grow dynamically without creating a new array and copying all existing elements.

CSHARP
int[] intArray = new int[10]; // Fixed-size array

#### Pros

Simple to use.
Good for fixed-size collections where the number of elements is known in advance.
Can be more efficient for certain operations, such as accessing elements by index.

#### Cons

Cannot grow or shrink after creation.
Requires allocation of a new array and copying data when capacity exceeds initial size.
Less flexible compared to List<T>.
List<T> in .NET

A List<T> is a dynamic array implemented as a class. It provides more flexibility than arrays by allowing the number of elements to grow or shrink dynamically during execution.

CSHARP
List<int> intList = new List<int>(); // Dynamic list

#### Pros

Dynamically resizable.
Provides methods for common operations like adding, removing, and searching elements.
Generally more convenient for most collection manipulation tasks due to its methods and built-in functionality.

#### Cons

Slightly less efficient in terms of memory usage when compared to arrays with known capacity.
Methods like Add and Remove can be slower than direct array access due to internal management overhead.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while both arrays and List<T> are used for storing collections in .NET, they differ significantly in terms of flexibility, performance, and ease of use. Arrays are fixed-size and more efficient when the size is known and constant, whereas List<T> offers dynamic resizing with additional functionality but comes at a slight cost in memory efficiency and overhead.

#### Common mistakes
include using arrays unnecessarily for collections that require frequent resizing or adding elements dynamically. Additionally, overlooking the built-in methods provided by List<T> can lead to suboptimal solutions where custom implementations of similar functionality might be necessary.

For follow-ups, an interviewer might ask about specific performance considerations, such as how List<T> handles resizing, or discuss scenarios where using arrays could be preferred over List<T>.

