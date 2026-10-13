---
type: "practice"
difficulty: "Intermediate"
---

# How do you invert a dictionary (swap keys and values), and what happens if values are not unique?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Membership and Collection Complexity](<../../../../TOOLS/Python/Python_Language/Membership_and_Collection_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To invert a dictionary in Python, swapping keys with their corresponding values, we need to handle cases where multiple keys may have the same value. This is crucial because dictionaries inherently map one key to one value; thus, when a value appears more than once, it implies that the inverted structure must account for this by mapping the common value back to all original keys.

#### Triage

The question asks us to invert a dictionary in Python and handle non-unique values. The core of the problem is understanding how to manage duplicate values effectively in the inversion process.

#### Assumptions
We are dealing with a standard Python dictionary, which maps keys to values.
Values may not be unique; we need to account for this during the inversion.

#### First Principles Solution

Let's break down the steps involved:

Input Dictionary: A typical example would be { 'a': 1, 'b': 2, 'c': 3 }.
Output Inverted Dictionary: We want to transform it into a dictionary where each value points back to its original keys.

We can achieve this in several ways:

Using a nested structure like a dictionary of lists.
Or, if the context allows, using sets for unique keys.

Here’s how we can implement both approaches:

#### Approach
1: Dictionary of Lists

This is more explicit and handles non-unique values gracefully by storing all keys that map to a given value in a list.

PYTHON
def invert_dict(d):
    inverted = {}
    for key, value in d.items():
        if value not in inverted:
            inverted[value] = [key]
        else:
            inverted[value].append(key)
    return inverted

# Example usage:
original_dict = {'a': 1, 'b': 2, 'c': 3, 'd': 2}
inverted_dict = invert_dict(original_dict)
print(inverted_dict)  # Output: {1: ['a'], 2: ['b', 'd'], 3: ['c']}

#### Approach
2: Dictionary of Sets

If the context allows for unique values to be stored in a set, we can use sets instead of lists.

PYTHON
def invert_dict_sets(d):
    inverted = {}
    for key, value in d.items():
        if value not in inverted:
            inverted[value] = {key}
        else:
            inverted[value].add(key)
    return inverted

# Example usage with the same original dictionary:
inverted_dict_sets = invert_dict_sets(original_dict)
print(inverted_dict_sets)  # Output: {1: {'a'}, 2: {'b', 'd'}, 3: {'c'}}

#### Closing Summary

Both methods effectively handle non-unique values and produce an inverted structure. The choice between a list or a set depends on the specific requirements of your application, such as whether order matters or if you want unique keys only.

#### Common Mistakes/Follow-Ups
Forgetting to Handle Non-Unique Values: If not considering duplicate values, the output will be incorrect.
Not Considering Time Complexity: Ensure that operations like in and append/add are efficient enough for your use case.
Choosing the Wrong Data Structure: Forcing a list when sets would suffice or vice versa can lead to unnecessary complexity.

**Code:**

```python
def invert_dict(d):
    inverted = {}
    for key, value in d.items():
        if value not in inverted:
            inverted[value] = [key]
        else:
            inverted[value].append(key)
    return inverted

def invert_dict_sets(d):
    inverted = {}
    for key, value in d.items():
        if value not in inverted:
            inverted[value] = {key}
        else:
            inverted[value].add(key)
    return inverted
```

**Example input / output:**

```text
inverted_dict = invert_dict(original_dict)
print(inverted_dict)  # Output: {1: ['a'], 2: ['b', 'd'], 3: ['c']}

inverted_dict_sets = invert_dict_sets(original_dict)
print(inverted_dict_sets)  # Output: {1: {'a'}, 2: {'b', 'd'}, 3: {'c'}}
```

**Data:**

```python
original_dict = {'a': 1, 'b': 2, 'c': 3, 'd': 2}
```

