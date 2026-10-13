---
type: "practice"
difficulty: "Basic"
---

# What is broadcasting in NumPy, and how does it let you add a 1D array to every row of a 2D array without a loop?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ NumPy Indexing](<../../../../TOOLS/Python/Python_Language/NumPy_Indexing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Broadcasting in NumPy is a powerful feature that allows operations on arrays with different shapes. This is particularly useful when we need to perform element-wise operations between arrays, such as adding a 1D array to every row of a 2D array without explicitly writing loops.

To understand broadcasting, let's first define the problem: you have a 2D array and a 1D array, and you want to add each element of the 1D array to every corresponding element in the rows of the 2D array. Normally, this would require nested loops, but with NumPy’s broadcasting, we can achieve this operation without explicit iteration.

Let's break down how broadcasting works step by step:

Shape Analysis: Broadcasting starts by comparing the shapes of the arrays involved.
Alignment and Padding: The smaller array is "broadcast" to match the shape of the larger one by replicating its elements along the appropriate dimensions.
Operation Execution: Once the shapes are aligned, element-wise operations can be performed.

Now, let's see a practical example using Python and NumPy.

**Code:**

```

import numpy as np

Define our 2D array and 1D array

matrix_2d = np.array([[1, 2, 3], [4, 5, 6]])

vector_1d = np.array([7, 8, 9])

print("Original 2D Array:") print(matrix_2d) print("\n1D Vector to be added to each row of the 2D array:") print(vector_1d)

Perform broadcasting

result = matrix_2d + vector_1d

print("\nResult after broadcasting addition:") print(result)

CODE
Copy

Input:
```python
import numpy as np

matrix_2d = np.array([[1, 2, 3],
                      [4, 5, 6]])

vector_1d = np.array([7, 8, 9])

Output:

CODE
Copy
Original 2D Array:
[[1 2 3]
 [4 5 6]]

1D Vector to be added to each row of the 2D array:
[7 8 9]

Result after broadcasting addition:
[[ 8 10 12]
 [11 13 15]]
```

**Example input / output:**

```

Input:

PYTHON
Copy
import numpy as np

matrix_2d = np.array([[1, 2, 3],
                      [4, 5, 6]])

vector_1d = np.array([7, 8, 9])

Output:

CODE
Copy
Original 2D Array:
[[1 2 3]
 [4 5 6]]

1D Vector to be added to each row of the 2D array:
[7 8 9]

Result after broadcasting addition:
[[ 8 10 12]
 [11 13 15]]
```

