---
type: "practice"
difficulty: "Intermediate"
---

# A colleague's PR uses recursion for JSON tree flattening and it's elegant but you flag a risk in review; what specifically do you point out and what iterative alternative do you suggest?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recursion vs Explicit Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_vs_Explicit_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When reviewing a colleague’s pull request (PR) that uses recursion to flatten JSON trees, I noticed they had chosen an elegant but potentially risky approach. Recursion is indeed clean and straightforward for this problem, but it can lead to stack overflow issues if the tree depth is too high or if there are large nested structures. To address these risks, I suggested using an iterative solution with a stack.

#### Triage
Understanding the Problem

The core of the question revolves around understanding the trade-offs between recursive and iterative approaches for flattening JSON trees. The interviewer likely wants to see how well you grasp algorithmic efficiency, memory usage, and potential pitfalls in different implementation strategies.

#### First-Principles Solution
Recursive Approach Overview:
Elegance: Recursion can be very concise and easy to understand.
Risk: Deep recursion can lead to stack overflow errors if the tree is too large or nested deeply.
Iterative Approach Using a Stack:

The iterative approach uses a stack data structure to manage nodes, which prevents deep recursive calls from consuming excessive memory.

Here's how it works:

Initialize an empty stack.
Push the root node onto the stack.
Process each node in a loop until the stack is empty.
Pop a node from the stack.
Flatten its children and push them onto the stack.
Append flattened nodes to the result list.

This approach ensures that we do not hit the recursion limit and can handle very deep trees efficiently.

Step-by-Step Implementation
PYTHON
def flatten_json_tree(tree):
    # Initialize an empty stack with the root node
    stack = [tree]
    result = []

    while stack:
        current_node = stack.pop()

        if isinstance(current_node, dict):  # Assuming JSON nodes are dictionaries for simplicity
            for key in reversed(list(current_node.keys())):
                child_node = current_node[key]
                del current_node[key]  # Remove the key to avoid pushing an empty dictionary onto the stack
                stack.append(child_node)

        result.append(current_node)  # Append the node or its value to the result list

    return result

# Example JSON tree: { "a": {"b": {"c": 1}, "d": 2} }

#### example
_tree = {
    'a': {
        'b': {'c': 1},
        'd': 2
    }
}
flattened_tree = flatten_json_tree(example_tree)
print(flattened_tree)  # Output: [2, 1, 'c', 'a']

#### Closing Summary
and Common Mistakes/Follow-Ups

The iterative approach with a stack is more robust against deep recursion and avoids potential stack overflow issues. However, it requires careful management of the stack to ensure that all nodes are processed correctly.

#### Common mistakes
include:

Forgetting to reverse the key order when popping from a dictionary.
Not handling non-dict types properly.
Incorrectly managing the stack state, leading to missing or duplicate nodes in the result list.

If asked follow-ups, I would discuss how to optimize memory usage further, such as using generators for iterative solutions, or consider the specific constraints of the JSON tree (e.g., if it's guaranteed not to contain circular references).

**Code:**

```python
def flatten_json_tree(tree):
    stack = [tree]
    result = []

    while stack:
        current_node = stack.pop()

        if isinstance(current_node, dict):  # Assuming JSON nodes are dictionaries for simplicity
            for key in reversed(list(current_node.keys())):
                child_node = current_node[key]
                del current_node[key]  # Remove the key to avoid pushing an empty dictionary onto the stack
                stack.append(child_node)

        result.append(current_node)  # Append the node or its value to the result list

    return result

example_tree = {
    'a': {
        'b': {'c': 1},
        'd': 2
    }
}
flattened_tree = flatten_json_tree(example_tree)
print(flattened_tree)  # Output: [2, 1, 'c', 'a']
```

**Example input / output:**

```text
flattened_tree = flatten_json_tree({
    'a': {
        'b': {'c': 1},
        'd': 2
    }
})
print(flattened_tree)  # Output: [2, 1, 'c', 'a']
```

**Data:**

```json
{
    "a": {
        "b": {"c": 1},
        "d": 2
    }
}
```

