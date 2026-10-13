---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a Python list and a tuple, and why might you prefer a tuple as a dictionary key for a feature combination?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Anagram Problems](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Anagram_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, I will explain the differences between Python lists and tuples, focusing on their use cases, particularly when considering tuples as keys in dictionaries for feature combinations.

TL;DR

A Python list is a mutable collection that can be modified after its creation. On the other hand, a tuple is an immutable collection that cannot be changed once created. Tuples are more appropriate as dictionary keys due to their immutability, which ensures they remain stable and hashable, making them ideal for representing unique feature combinations.

#### Triaging the Question

The question primarily probes your understanding of fundamental Python data structures—specifically lists and tuples—and how these differences affect their usage in specific scenarios. Key assumptions include:

You understand the basic operations on both data types.
You know that dictionaries require keys to be hashable.
You are familiar with the concept of immutability.

#### First-Principles Solution

Mutability and Operations:

A list is mutable, meaning elements can be added, removed, or modified after its creation. Example: my_list = [1, 2, 3]; my_list[0] = 4.
A tuple is immutable, so once created, its contents cannot change. Example: my_tuple = (1, 2, 3); attempting to modify it will raise an error.

Hashability and Keys:

Tuples are hashable, which means they can be used as dictionary keys. Lists are not hashable because their mutability makes them unpredictable.
For example, the following code snippet would work with a tuple but fail with a list:
PYTHON
dict_key_tuple = {(1, 2): "Feature Combination A"}
# This works: print(dict_key_tuple[(1, 2)])
dict_key_list = {[1, 2]: "Feature Combination B"}  # Raises TypeError: unhashable type: 'list'

Use Cases:

Tuples are often used to represent data that should not change, such as feature combinations or coordinates.
Lists are useful when you need a dynamic collection of items where changes might be frequent.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key difference lies in mutability: tuples allow them to be used as dictionary keys due to their immutability. Always prefer tuples for feature combinations or any scenario requiring stable, hashable keys.

#### Common Mistakes

Confusing the use cases of lists and tuples.
Assuming that mutable collections like lists can be used as dictionary keys without causing issues.
Overlooking the importance of hashability in choosing data structures for dictionary keys.

#### Example
Dry-Run
PYTHON
# Valid usage with a tuple as a key
feature_dict = {(1, 2): "Feature Combination A", (3, 4): "Feature Combination B"}
print(feature_dict[(1, 2)])  # Output: Feature Combination A

# Invalid usage with a list as a key
try:
    feature_dict = {[1, 2]: "Feature Combination C"}
except TypeError as e:
    print(e)  # Output: unhashable type: 'list'

**Code:**

```python
my_list = [1, 2, 3]
my_tuple = (1, 2, 3)

# Modifying a list is allowed
my_list[0] = 4

# Attempting to modify a tuple will raise an error
try:
    my_tuple[0] = 4
except TypeError as e:
    print(e)  # Output: 'tuple' object does not support item assignment
```

