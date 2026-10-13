---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between shallow and deep copy when duplicating a nested config dictionary for a hyperparameter sweep.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Recursion vs Explicit Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_vs_Explicit_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with nested configurations in Python, understanding the difference between shallow and deep copies is crucial. A shallow copy creates a new object but references the original elements within it, whereas a deep copy duplicates both the container and its contents entirely.

In this context, let's delve into how these concepts apply to duplicating a nested config dictionary for a hyperparameter sweep.

#### Triage
What Are We Copying?

When you have a nested configuration dictionary used in a hyperparameter sweep, you often want to duplicate this dictionary to avoid mutating the original. The key is understanding whether you need to modify just the top-level elements or all levels of nesting as well.

#### Assumptions
The config dictionary has multiple levels of nested keys and values.
We are working with Python’s built-in dict type for simplicity.

#### First-Principles Solution
Shallow Copy vs. Deep Copy
Shallow Copy

A shallow copy creates a new dictionary but references the same objects in its sub-dictionaries. Essentially, it only copies one level deep. If you modify a mutable value at any deeper level within the copied dictionary, it will also affect the original.

PYTHON
import copy

# Original nested configuration dictionary
config = {
    "model": {"name": "linear", "params": {"learning_rate": 0.1}},
    "data": {"batch_size": 32}
}

# Shallow copy
shallow_config = copy.copy(config)

print("Original:", config)
print("Shallow Copy:", shallow_config)
Deep Copy

A deep copy duplicates every object in the original dictionary, including sub-dictionaries and lists. This means any modifications to mutable objects within the copied dictionary do not affect the original.

PYTHON
# Deep copy
deep_config = copy.deepcopy(config)

print("Deep Copy:", deep_config)

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, shallow copies are useful when you want a quick duplication without worrying about the deeper levels of the data structure. However, they can lead to unexpected side effects in hyperparameter sweeps if mutable values within sub-dictionaries get modified.

Deep copies ensure that any modifications do not affect the original configuration, which is crucial for maintaining consistency and avoiding unintended side effects during hyperparameter tuning.

#### Common Mistakes
Assuming all dictionaries are deep copied automatically: Shallow copies can still lead to issues with mutable objects in nested structures.
Not understanding the implications of shallow vs. deep copying on mutable data types (like lists or other dictionaries).

#### Follow-Ups
How would you handle a case where the config dictionary contains both immutable and mutable values?
What are some scenarios where shallow copying might be preferred over deep copying?

**Comparison:**

| Operation | Shallow Copy                          | Deep Copy                             |
|-----------|---------------------------------------|---------------------------------------|
| Creates   | A new object with references          | A completely new object               |
| Modifications | Modify mutable elements in the copy can affect the original | Any modifications to mutable elements do not affect the original |

**Code:**

```

import copy

Original nested configuration dictionary

config = { "model": {"name": "linear", "params": {"learning_rate": 0.1}}, "data": {"batch_size": 32} }

Shallow copy

shallow_config = copy.copy(config)

print("Original:", config) print("Shallow Copy:", shallow_config)

Modifying a mutable value in the shallow copied dictionary

shallow_config["model"]["params"]["learning_rate"] = 0.5

print("\nAfter modifying learning rate:") print("Original:", config) # Original is unchanged print("Shallow Copy:", shallow_config)

CODE
Copy

```python
# Deep copy
deep_config = copy.deepcopy(config)

print("Deep Copy:", deep_config)

# Modifying a mutable value in the deeply copied dictionary
deep_config["model"]["params"]["learning_rate"] = 0.5

print("\nAfter modifying learning rate:")
print("Original:", config)  # Original remains unchanged due to deep copying
print("Deep Copy:", deep_config)
```

**Example input / output:**

```text
import copy

# Original nested configuration dictionary
config = {
    "model": {"name": "linear", "params": {"learning_rate": 0.1}},
    "data": {"batch_size": 32}
}

# Shallow copy
shallow_config = copy.copy(config)

print("Original:", config)
print("Shallow Copy:", shallow_config)

# Modifying a mutable value in the shallow copied dictionary
shallow_config["model"]["params"]["learning_rate"] = 0.5

print("\nAfter modifying learning rate:")
print("Original:", config)  # Original is unchanged
print("Shallow Copy:", shallow_config)

# Deep copy
deep_config = copy.deepcopy(config)

print("\nDeep Copy:", deep_config)

# Modifying a mutable value in the deeply copied dictionary
deep_config["model"]["params"]["learning_rate"] = 0.5

print("\nAfter modifying learning rate:")
print("Original:", config)  # Original remains unchanged due to deep copying
print("Deep Copy:", deep_config)
```

**Data:**

```json
{
    "config": {
        "model": {"name": "linear", "params": {"learning_rate": 0.1}},
        "data": {"batch_size": 32}
    }
}
```

**Flow:**

1. Yes
2. No
3. Original Dictionary
4. Is Element Mutable?
5. Shallow Copy
6. Modify Original May AffectCopied
7. Deep Copy
8. Safe to Modify WithoutAffecting Original

