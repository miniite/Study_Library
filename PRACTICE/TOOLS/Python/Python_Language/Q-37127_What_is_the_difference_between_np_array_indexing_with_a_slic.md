---
type: "practice"
difficulty: "Basic"
---

# What is the difference between np.array indexing with a slice versus with a boolean mask, and when do you use each?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ NumPy Indexing](<../../../../TOOLS/Python/Python_Language/NumPy_Indexing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with NumPy arrays in Python, understanding how to index them using slices versus boolean masks is crucial. In essence, slicing allows you to select elements based on their position, while boolean masking lets you select elements based on a condition. Let's dive into the differences and when to use each.

#### Triage

The question probes your knowledge of NumPy array indexing techniques in Python. It asks for an explanation of the fundamental differences between using slices and boolean masks to index arrays. Additionally, it seeks examples of situations where you would prefer one method over the other.

#### First-Principles Solution
Slicing vs. Boolean Masking
Slicing:
Definition: A slice is a range of indices that can be used to select contiguous elements in an array.
Usage Example: arr[start:stop:step] where start, stop, and step define the beginning, end, and step size respectively.
Boolean Masking:
Definition: A boolean mask is a NumPy array of booleans (True or False) used to filter elements based on conditions.
Usage Example: arr[mask] where mask is an array of the same length as arr, with True values indicating which indices should be selected.
When to Use Each

Slicing:

Prefer slicing when you need to access a range of elements in your array, especially if they are contiguous.

#### Example
Accessing all even-indexed elements from an array. In this case, arr[::2] would be used.

Boolean Masking:

Use boolean masking when you want to filter elements based on specific conditions that do not necessarily form a continuous range.

#### Example
Selecting elements greater than a certain value in the array, such as arr[arr > threshold].

#### Closing Summary
and Common Mistakes

Slicing is more straightforward for accessing contiguous ranges of elements, while boolean masking offers flexibility by allowing condition-based filtering. Both techniques are powerful tools in the NumPy arsenal, but their choice depends on the specific use case.

#### Common Mistakes
Overusing Slicing: Don’t slice unnecessarily; it can make your code harder to read if you’re not selecting a continuous range.
Misunderstanding Boolean Masks: Sometimes people might think they need to iterate over an array using loops when boolean masking would be more efficient and cleaner.

#### Example

Let's consider an example with an array arr = np.array([1, 2, 3, 4, 5]):

Slicing Example:
PYTHON
even_elements = arr[::2]  # Output: array([1, 3, 5])
Boolean Masking Example:
PYTHON
threshold = 3
above_threshold_elements = arr[arr > threshold]  # Output: array([4, 5])

**Code:**

```python
import numpy as np

# Example array
arr = np.array([1, 2, 3, 4, 5])

# Slicing example
even_elements = arr[::2]
print(even_elements)  # Output: [1 3 5]

# Boolean masking example
threshold = 3
above_threshold_elements = arr[arr > threshold]
print(above_threshold_elements)  # Output: [4 5]
```

**Example input / output:**

```text
# Slicing input->output
arr = np.array([1, 2, 3, 4, 5])
print(arr[::2])  # Output: array([1, 3, 5])

# Boolean masking input->output
threshold = 3
print(arr[arr > threshold])  # Output: array([4, 5])
```

**Data:**

```json
{
  "arr": [1, 2, 3, 4, 5],
  "even_elements": [1, 3, 5],
  "threshold": 3,
  "above_threshold_elements": [4, 5]
}
```

