---
type: "practice"
difficulty: "Basic"
---

# How would you compute a confusion matrix from arrays of predictions and true labels, and what is the time complexity?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Complexity of Encodings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Complexity_of_Encodings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a confusion matrix, we first need to understand its purpose: it's a table used to describe the performance of a classification model. It breaks down predicted versus actual outcomes into a square matrix where rows represent true classes and columns represent predicted classes.

The question is essentially asking how to map predictions and true labels into this matrix format efficiently. We'll start by assuming we have two arrays: one for predictions (preds) and another for true labels (true_labels). The length of both arrays will be the same, as they correspond to individual data points in our dataset.

To build a confusion matrix from these, let's break down the steps:

Define the number of classes.
Initialize a matrix with zeros, where each dimension corresponds to the class count.
Iterate through each prediction and true label pair, updating the appropriate cell in the matrix based on the predicted vs actual match.

The time complexity of this operation will be O(n), where n is the number of data points (the length of preds or true_labels).

Now let's go through a detailed step-by-step solution.

**Code:**

```python
# Sample input arrays for predictions and true labels
preds = [1, 2, 3, 4, 5]
true_labels = [2, 1, 3, 4, 2]

# Number of classes (assuming the highest label is the class count)
num_classes = max(max(preds), max(true_labels))

# Initialize a confusion matrix with zeros
confusion_matrix = [[0 for _ in range(num_classes)] for _ in range(num_classes)]

# Iterate through each prediction and true label pair, updating the confusion matrix
for pred, true_label in zip(preds, true_labels):
    confusion_matrix[true_label - 1][pred - 1] += 1

confusion_matrix

This code snippet initializes a confusion matrix of size num_classes x num_classes with all elements set to zero. It then iterates through the predictions and updates the corresponding cell in the matrix based on the true label.
```

**Example input / output:**

```

Input:

PYTHON
Copy
preds = [1, 2, 3, 4, 5]
true_labels = [2, 1, 3, 4, 2]

Output:

PLAINTEXT
Copy
[[0, 0, 0, 0, 0],
 [1, 0, 1, 0, 0],
 [0, 0, 1, 1, 0]]

This matrix shows that the first prediction was a 2 (which is true for the second data point only), and so on.
```

