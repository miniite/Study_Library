---
type: "practice"
difficulty: "Intermediate"
---

# A shared utility function mutates the list it's passed and a caller was surprised. Discuss defensive copying vs documenting mutation vs returning new objects as an API design choice.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Mutability](<../../../../TOOLS/Python/Python_Language/Mutability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this Python-based utility function issue, we need to understand the problem context and explore different design choices. The core of the problem is that a shared utility function directly modifies the list passed as an argument, leading to unexpected behavior in the calling code. As an intermediate-level engineer, you should consider defensive copying, documenting mutation, or returning new objects as viable solutions.

Triaging the Problem

The question revolves around how to handle mutable data structures (like lists) in Python functions and ensure that function calls do not lead to unintended side effects. The key is to determine whether the utility function should modify its input directly or return a modified copy, depending on the API design principles you want to follow.

#### Assumptions
Mutability: Lists are mutable in Python, meaning they can be changed after creation.
Function Behavior: We need to ensure that the function's behavior is clear and predictable for its callers.
Memory Efficiency: Consideration should be given to memory usage, especially when dealing with large data structures.

#### First-Principles Solution
Defensive Copying

In Python, defensive copying involves making a new copy of an object (or parts of it) before modifying the copy. This ensures that the original list remains unchanged.

Code Example:

PYTHON
def process_list(defensive_copy):
    # Process the list and return a modified version
    processed_list = [element * 2 for element in defensive_copy]
    return processed_list

original_list = [1, 2, 3, 4]
new_list = process_list(original_list)
print("Original List:", original_list)  # Output: Original List: [1, 2, 3, 4]
print("Processed List:", new_list)     # Output: Processed List: [2, 4, 6, 8]
Documenting Mutation

Another approach is to document the function's behavior clearly. If a caller needs a modified version of the list, they should be aware that passing it directly will mutate it.

Code Example:

PYTHON
def process_list(mutable_list):
    """
    Mutates the input list by doubling each element.
    """
    for i in range(len(mutable_list)):
        mutable_list[i] *= 2

original_list = [1, 2, 3, 4]
process_list(original_list)
print("Processed List:", original_list)  # Output: Processed List: [2, 4, 6, 8]
Returning New Objects

Returning a new list with the desired changes is another common approach. This ensures that the original list remains unchanged.

Code Example:

PYTHON
def process_list(new_objects):
    processed_list = [element * 2 for element in new_objects]
    return processed_list

original_list = [1, 2, 3, 4]
new_list = process_list(original_list)
print("Original List:", original_list)  # Output: Original List: [1, 2, 3, 4]
print("Processed List:", new_list)      # Output: Processed List: [2, 4, 6, 8]

#### Closing Summary
and Common Mistakes
Defensive Copying is a safe approach but may consume more memory, especially for large lists.
Documenting Mutation relies on clear documentation and can lead to confusion if not properly documented.
Returning New Objects keeps the original list intact but requires an additional function call.

#### Common mistakes
include:

Failing to document mutation clearly, leading to bugs in downstream code.
Overusing defensive copying for performance-critical code without considering memory implications.
Not understanding the differences between these approaches and choosing the wrong one based on context.
Final Thoughts

In practice, the choice depends on the specific requirements of your application. For safety and clarity, using defensive copies or returning new objects is often preferred over directly mutating inputs. However, documenting mutation effectively can be a viable alternative if memory usage is a concern and proper documentation practices are followed.

**Comparison:**

| Approach          | Memory Usage       | Predictability | Downstream Impact |
|-------------------|--------------------|----------------|------------------|
| Defensive Copying | Higher (for large lists) | High         | None             |
| Documented Mutation | Low      | Medium        | Potential bugs    |
| Returning New Object | Low     | Medium        | Additional function call |

**Code:**

```python
# Example code for defensive copying
def process_list(defensive_copy):
    processed_list = [element * 2 for element in defensive_copy]
    return processed_list

original_list = [1, 2, 3, 4]
new_list = process_list(original_list)
print("Original List:", original_list)  # Output: Original List: [1, 2, 3, 4]
print("Processed List:", new_list)     # Output: Processed List: [2, 4, 6, 8]

# Example code for mutating the input directly
def process_list(mutable_list):
    """
    Mutates the input list by doubling each element.
    """
    for i in range(len(mutable_list)):
        mutable_list[i] *= 2

original_list = [1, 2, 3, 4]
process_list(original_list)
print("Processed List:", original_list)  # Output: Processed List: [2, 4, 6, 8]

# Example code for returning a new object
def process_list(new_objects):
    processed_list = [element * 2 for element in new_objects]
    return processed_list

original_list = [1, 2, 3, 4]
new_list = process_list(original_list)
print("Original List:", original_list)  # Output: Original List: [1, 2, 3, 4]
print("Processed List:", new_list)      # Output: Processed List: [2, 4, 6, 8]
```

**Example input / output:**

```text
Input:
original_list = [1, 2, 3, 4]

Output with Defensive Copying:
Original List: [1, 2, 3, 4]
Processed List: [2, 4, 6, 8]

Output with Mutating Input Directly:
Processed List: [2, 4, 6, 8]

Output with Returning New Object:
Original List: [1, 2, 3, 4]
Processed List: [2, 4, 6, 8]
```

**Data:**

```json
{
  "original_list": [1, 2, 3, 4],
  "processed_list_defensive_copy": [2, 4, 6, 8],
  "processed_list_mutate_input": [2, 4, 6, 8],
  "new_objects_processed_list": [2, 4, 6, 8]
}
```

**Flow:**

1. Yes
2. No
3. Function Call
4. Input List
5. Process List
6. Is Input Mutable?
7. Mutate Input & Return
8. Create Copy, Process, ReturnNew List

