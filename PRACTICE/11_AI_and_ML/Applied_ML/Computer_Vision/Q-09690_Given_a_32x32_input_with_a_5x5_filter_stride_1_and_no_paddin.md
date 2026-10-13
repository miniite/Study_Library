---
type: "practice"
difficulty: "Intermediate"
---

# Given a 32x32 input with a 5x5 filter, stride 1 and no padding, what will be the output feature map size?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CNN Fundamentals](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/CNN_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine the output feature map size when applying a 5x5 filter to a 32x32 input with a stride of 1 and no padding, we need to understand the basic principles of convolutional operations in computer vision.

#### Triage

The question is essentially asking us to calculate the dimensions of the output feature map after performing a convolution operation. The key parameters are:

Input size: 32x32 pixels.
Filter size: 5x5.
Stride: 1.
Padding: None.

#### First-Principles Solution

The formula to calculate the output size (O) for an input of size (I), filter size (F), and stride (S) with no padding is given by: [ O = \frac{I - F}{S} + 1 ]

Let's break down this calculation step-by-step.

Step 1: Applying the Formula to Width
Input width (I_w): 32.
Filter width (F_w): 5.
Stride (S_w): 1.

Using the formula: [ O_w = \frac{32 - 5}{1} + 1 = \frac{27}{1} + 1 = 27 + 1 = 28 ]

Step 2: Applying the Formula to Height
Input height (I_h): 32.
Filter height (F_h): 5.
Stride (S_h): 1.

Using the same formula: [ O_h = \frac{32 - 5}{1} + 1 = \frac{27}{1} + 1 = 27 + 1 = 28 ]

Thus, the output feature map will have dimensions of 28x28.

#### Closing Summary
and Common Mistakes

The final answer is that the output feature map size will be 28x28. A common mistake people make is forgetting to add 1 in the formula, which leads them to believe the result should be simply (32 - 5 = 27). Always remember to include this "+1" as it accounts for the initial position of the filter.

Depth + Language Match the Level

For a fresher or an intermediate ML Engineer, understanding the concept and applying a straightforward formula is sufficient. It's important to grasp why we add 1 in the calculation; it ensures that the first filter can also be applied at the beginning of the input without losing any information.

**Code:**

```python
def calculate_output_size(input_size, filter_size, stride):
    return (input_size - filter_size) // stride + 1

output_width = calculate_output_size(32, 5, 1)
output_height = calculate_output_size(32, 5, 1)

print(f"Output Feature Map Size: {output_width}x{output_height}")
```

**Example input / output:**

```
Sample Calculation

input_size = 32 filter_size = 5 stride = 1

Calculate output dimensions

output_width = (input_size - filter_size) // stride + 1 output_height = (input_size - filter_size) // stride + 1

print(f"Output Feature Map Size: {output_width}x{output_height}")

CODE
Copy
Output:

Output Feature Map Size: 28x28
```

