---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a set and a multiset, and when would you reach for each?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backtracking](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Backtracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll explore the fundamental differences between sets and multisets in data structures, understanding their core concepts and scenarios where they are most useful.

#### Triaging the Question

The key difference lies in how each structure handles unique elements:

Set: A collection of unique elements. Each element appears only once.
Multiset: A collection that allows multiple occurrences of the same element.

Essentially, sets ensure uniqueness by design, whereas multisets allow for frequency tracking.

#### First Principles Solution
Sets
Uniqueness Guarantee:
Every element in a set is unique.
Operations:
insert: Adds an element if it's not already present.
remove: Removes the first occurrence of an element.
contains: Checks for membership without duplicates.
Multisets
Frequency Tracking:
Elements can appear multiple times, and their frequency is tracked.
Operations:
insert: Adds one more instance of the element.
remove: Reduces the count by one if available; if zero, removes it.
count: Returns the number of occurrences of an element.

#### Common Mistakes/Follow-Ups

Misunderstanding Uniqueness:

A common mistake is thinking that sets always require custom implementations for uniqueness checks, when built-in language features like hash maps can be used effectively.

Ignoring Frequency Tracking:

Developers might overlook the importance of frequency tracking in multisets, leading to inefficient or incorrect usage patterns.

Scalability Concerns:

Both structures have their trade-offs regarding time and space complexity. For instance, sets with hash-based implementations offer average O(1) operations but may degrade to O(n) under certain conditions like hash collisions.

#### Closing Summary

Sets are ideal when you need a collection of unique elements without worrying about their count. Multisets, on the other hand, are necessary when frequency tracking is essential or when you need to handle duplicate data efficiently.

**Comparison:**

```
Set Operations	Multiset Operations
insert	increment_count
remove	decrement_count
contains	count
```

**Code:**

```python
# Example Python code for sets and multisets using collections.Counter

from typing import Set, Dict
import collections

def use_set() -> None:
    s = set()
    elements = [1, 2, 3, 4, 5, 1, 2]
    for elem in elements:
        if elem not in s:
            s.add(elem)
    print(s)  # Output: {1, 2, 3, 4, 5}

def use_multiset() -> None:
    m = collections.Counter()
    elements = [1, 2, 3, 4, 5, 1, 2]
    for elem in elements:
        m[elem] += 1
    print(m)  # Output: Counter({1: 2, 2: 2, 3: 1, 4: 1, 5: 1})
```

**Example input / output:**

```text
Input for set:
- Elements: [1, 2, 3, 4, 5, 1, 2]

Output for set:
- Unique elements: {1, 2, 3, 4, 5}

Input for multiset:
- Elements: [1, 2, 3, 4, 5, 1, 2]

Output for multiset:
- Element counts: Counter({1: 2, 2: 2, 3: 1, 4: 1, 5: 1})
```

**Data:**

```json
{
  "elements_set": [1, 2, 3, 4, 5, 1, 2],
  "elements_multiset": [1, 2, 3, 4, 5, 1, 2]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Set
6. Is element in set?
7. Add element
8. Remove element if present
9. Multiset
10. Increment count of element?
11. Update count
12. Insert element with initialcount 1

