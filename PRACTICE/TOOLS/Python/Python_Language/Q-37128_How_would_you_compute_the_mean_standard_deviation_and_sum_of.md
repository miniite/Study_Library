---
type: "practice"
difficulty: "Basic"
---

# How would you compute the mean, standard deviation, and sum of a NumPy array along a specific axis, and what does axis=0 versus axis=1 mean?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Normalization with NumPy](<../../../../TOOLS/Python/Python_Language/Vector_Normalization_with_NumPy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the mean, standard deviation, and sum of a NumPy array along a specific axis, we need to understand how axes are defined in NumPy arrays and then use appropriate functions from the NumPy library. Let's break this down step by step.

#### Triaging the Question

The question is asking about operations (mean, standard deviation, and sum) on a NumPy array using different axes (axis=0 and axis=1). The interviewer wants to ensure you understand how these operations work across the rows or columns of your data.

#### First-Principles Solution

NumPy arrays are multidimensional arrays where each dimension is called an axis. For a 2D array, axis=0 refers to the vertical direction (rows), and axis=1 refers to the horizontal direction (columns).

Mean: The mean along an axis calculates the average of elements in that direction.
Standard Deviation: This measures how spread out numbers are from the mean.
Sum: Summing up values along an axis gives us a single scalar value for each row or column.

#### Example
Code and Explanation

Here’s how we can perform these operations using NumPy:

PYTHON
import numpy as np

# Create a sample 2D array
data = np.array([[1, 2, 3], [4, 5, 6]])
print("Sample Data:\n", data)

# Compute mean along axis=0 (rows)
mean_axis_0 = np.mean(data, axis=0)
print("\nMean along Axis 0 (Rows):\n", mean_axis_0)

# Compute mean along axis=1 (columns)
mean_axis_1 = np.mean(data, axis=1)
print("\nMean along Axis 1 (Columns):\n", mean_axis_1)

# Compute standard deviation along both axes
std_dev_axis_0 = np.std(data, axis=0)
std_dev_axis_1 = np.std(data, axis=1)
print("\nStandard Deviation along Axis 0 (Rows):\n", std_dev_axis_0)
print("Standard Deviation along Axis 1 (Columns):\n", std_dev_axis_1)

# Compute sum along both axes
sum_axis_0 = np.sum(data, axis=0)
sum_axis_1 = np.sum(data, axis=1)
print("\nSum along Axis 0 (Rows):\n", sum_axis_0)
print("Sum along Axis 1 (Columns):\n", sum_axis_1)

#### Closing Summary
and Common Mistakes

This example demonstrates how to use NumPy functions to compute the mean, standard deviation, and sum of a 2D array. Key points are:

axis=0 processes rows.
axis=1 processes columns.

#### Common mistakes
include:

Forgetting that default axis is 0 (rows).
Misunderstanding what each function returns based on the specified axis.
Final Thoughts

Understanding axes in NumPy is crucial for multidimensional array operations. Always double-check your axis parameter to ensure it matches the data structure you are working with.

