---
type: "practice"
difficulty: "Basic"
---

# How do you iterate over a dictionary's keys, values, and key-value pairs? Show all three.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Membership and Collection Complexity](<../../../../TOOLS/Python/Python_Language/Membership_and_Collection_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with dictionaries in Python, there are three common ways to iterate over their keys, values, and key-value pairs. Let’s explore each method step by step using first principles.

PYTHON
# Example dictionary

#### example
_dict = {"a": 1, "b": 2, "c": 3}

# Iterating over keys
for key in example_dict.keys():
    print(f"Key: {key}")

# Iterating over values
for value in example_dict.values():
    print(f"Value: {value}")

# Iterating over key-value pairs
for key, value in example_dict.items():
    print(f"Key-Value Pair: ({key}, {value})")

The code checks out for all three iterations.

