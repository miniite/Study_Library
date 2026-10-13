---
type: "practice"
difficulty: "Basic"
---

# How would you prompt Copilot to write a confusion-matrix heatmap and verify the axis labels aren't swapped (predicted vs actual)?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Verifying AI Generated ML Code](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Verifying_AI_Generated_ML_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To prompt Copilot to generate a confusion matrix heatmap and ensure that the predicted and actual axes are correctly labeled, you would need to provide clear instructions in natural language along with specific code snippets or data structures. The key is to be precise about what you want Copilot to do, including any validations or checks for correctness.

#### Triaging the Question

The question asks for a method to prompt Copilot (or similar AI tools) to generate a confusion matrix heatmap and verify that the predicted versus actual labels are correctly aligned. This involves two main steps: generating the heat map and ensuring the axes are not swapped.

Key Assumptions
You have data ready in some structured format, typically a 2D array representing the confusion matrix.
Copilot has the capability to interpret natural language prompts and execute Python code.
The goal is both to generate the heatmap and to check for axis label correctness.
Step-by-Step Solution from First Principles
Define the Confusion Matrix: Start by defining your data structure, which should be a 2D array where rows represent actual classes and columns represent predicted classes.
Generate the Heatmap: Use libraries like seaborn or matplotlib to create the heatmap.
Verify Axis Labels: Ensure that the labels are correctly aligned as 'Actual' on the y-axis and 'Predicted' on the x-axis.
Code Example
PYTHON
import seaborn as sns
import numpy as np

# Sample confusion matrix data (2D array)
confusion_matrix = np.array([
    [10, 5, 3],
    [2, 15, 4],
    [7, 6, 18]
])

# Prompt Copilot to generate the heatmap
copilot_prompt = """
def generate_confusion_matrix_heatmap(confusion_matrix):
    # Verify axis labels are correct: Actual on y-axis and Predicted on x-axis
    if confusion_matrix.shape[0] != confusion_matrix.shape[1]:
        raise ValueError("Confusion matrix is not square. Axis labels might be swapped.")

    # Create a heatmap using seaborn
    sns.heatmap(confusion_matrix, annot=True, fmt="d", cmap='YlGnBu')
    plt.xlabel('Predicted')
    plt.ylabel('Actual')
    plt.title('Confusion Matrix Heatmap')
    plt.show()

# Test the function with sample data
generate_confusion_matrix_heatmap(confusion_matrix)
"""

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake 1: Forgetting to check if the confusion matrix is square, which can lead to misaligned axes.
Common Mistake 2: Not providing clear instructions in your prompt, leading Copilot to generate incorrect code.

#### Follow-Up
Question: How would you handle non-square matrices or different types of data structures?

Ensure that your prompt is clear and includes all necessary checks. By doing so, you can effectively leverage AI tools like Copilot for tasks like generating and validating confusion matrix heatmaps.

**Comparison:**

```
Predicted	0	1	2
Actual	10	5	3
Actual	2	15	4
Actual	7	6	18
```

**Code:**

```python
import seaborn as sns
import numpy as np

confusion_matrix = np.array([
    [10, 5, 3],
    [2, 15, 4],
    [7, 6, 18]
])

def generate_confusion_matrix_heatmap(confusion_matrix):
    if confusion_matrix.shape[0] != confusion_matrix.shape[1]:
        raise ValueError("Confusion matrix is not square. Axis labels might be swapped.")

    sns.heatmap(confusion_matrix, annot=True, fmt="d", cmap='YlGnBu')
    plt.xlabel('Predicted')
    plt.ylabel('Actual')
    plt.title('Confusion Matrix Heatmap')
    plt.show()
```

**Example input / output:**

```text
# Test the function with sample data
generate_confusion_matrix_heatmap(confusion_matrix)
```

**Data:**

```python
confusion_matrix = np.array([
    [10, 5, 3],
    [2, 15, 4],
    [7, 6, 18]
])
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Confusion Matrix Square?
5. Generate Heatmap
6. Error: Axis Labels Might BeSwapped
7. Display Heatmap with CorrectAxis Labels

