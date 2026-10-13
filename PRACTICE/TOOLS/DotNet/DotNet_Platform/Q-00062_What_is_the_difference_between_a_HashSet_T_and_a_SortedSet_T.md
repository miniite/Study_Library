---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a HashSet<T> and a SortedSet<T>, and what are their lookup costs?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ LINQ and Collections](<../../../../TOOLS/DotNet/DotNet_Platform/LINQ_and_Collections.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Both HashSet<T> and SortedSet<T> are collections in .NET designed to store unique elements, but they serve different purposes and have distinct characteristics. Let's explore the differences between them and their respective lookup costs.

#### Triage

The question is asking you to differentiate between two collection types in C# (HashSet<T> and SortedSet<T>) and understand how these differences impact their performance, particularly with respect to lookup operations. The interviewer might be probing your understanding of .NET collections and your ability to articulate the trade-offs between different data structures.

#### Assumptions
HashSet<T>: A collection that stores unique elements without any particular order.
SortedSet<T>: A collection that also stores unique elements but maintains them in a sorted order based on their natural ordering or a specified comparer.

#### First-Principles Solution
HashSet<T>
Data Structure: Typically implemented as an array of linked lists (using hash codes).
Lookup Cost: Average O(1) time complexity. Since each element is hashed to determine its position, the lookup operation can be very fast.
Use Case: Ideal for scenarios where you need fast lookups and don't care about the order of elements.
SortedSet<T>
Data Structure: Usually implemented as a balanced binary search tree (such as an AVL tree or Red-Black tree).
Lookup Cost: Average O(log n) time complexity. The elements are maintained in sorted order, which allows for efficient searching and iteration.
Use Case: Suitable when you need to maintain the collection in sorted order and perform operations like min/max queries.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, HashSet<T> offers faster lookup times but sacrifices ordering, making it suitable for scenarios where performance is critical. On the other hand, SortedSet<T> provides ordered elements with slightly slower lookups, ideal for cases requiring sorted access and operations.

#### Common mistakes
to avoid include:

Confusing the order of elements in both collections.
Overlooking the trade-offs between lookup speed and ordering requirements.
Not considering the specific performance characteristics when choosing a collection type.

During follow-ups, you might be asked about edge cases or other related topics such as memory usage, insertion performance, and specific use cases where one might prefer HashSet<T> over SortedSet<T>.

**Comparison:**

```
Collection	Data Structure	Lookup Cost (Average)	Use Case
HashSet<T>	Hash Table	O(1)	Fast lookups, unordered elements
SortedSet<T>	Balanced BST	O(log n)	Ordered elements, sorted access
```

**Code:**

```csharp
// Example of using HashSet<T>
var hashSet = new HashSet<int>();
hashSet.Add(1);
hashSet.Add(2);
hashSet.Add(3);

// Example of using SortedSet<T>
using System.Collections.Generic;
var sortedSet = new SortedSet<int>(new List<int> { 3, 2, 1 });
```

