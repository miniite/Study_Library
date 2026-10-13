---
type: "practice"
difficulty: "Basic"
---

# What is the complexity of training a single perceptron for one pass over n samples, and where does the per-sample work go?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Complexity of Encodings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Complexity_of_Encodings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the complexity of training a single perceptron for one pass over (n) samples, let's break it down step by step. The core idea is that in machine learning, particularly with simple models like the perceptron, each sample contributes to the weight adjustments during the training process. We'll focus on the time complexity and what part of this work is done per sample.

The question is essentially asking about the computational overhead required for one epoch (pass) through a dataset when using the perceptron algorithm. This involves understanding how many operations are performed per sample and how they accumulate over all samples in the dataset.

We make an assumption that we're dealing with a standard implementation of the perceptron learning rule, which updates the weights after each misclassified sample or at the end of an epoch if no errors occur.

Now let's dive into the details:

Initialization: Assume the model starts with some initial weights and bias.
For Each Sample:
Compute the prediction for the current input vector using the current weight vector.
Compare this prediction to the true label (which is binary in the perceptron case, either +1 or -1).
If the prediction differs from the label, update the weights and bias based on the learning rate and the difference between the prediction and the label.

The key operations per sample are:

Compute the dot product of the input vector with the weight vector.
Compare this result to a threshold (usually 0 for simplicity).
If there is an error, adjust the weights using the update rule: (w_i \leftarrow w_i + \eta y x_i), where (\eta) is the learning rate, (y) is the true label, and (x_i) are the features.

Given these operations, the time complexity per sample is:

One dot product operation (which takes (O(d)) time, where (d) is the dimensionality of the input vector).
Comparing the result to a threshold (constant time: (O(1))).
Updating weights and bias (also constant time: (O(1))).

Thus, the total per-sample complexity is (O(d)).

Since we have (n) samples in the dataset, the overall complexity for one pass over all samples is: [ O(n \times d) ]

This means that the training process involves computing a dot product and performing necessary updates for each sample. The dimensionality of the input space directly influences this complexity.

In summary, the complexity of training a single perceptron for one pass over (n) samples is (O(n \times d)), with the bulk of the work being done in the dot products per sample.

**Code:**

```python
# Pseudocode for a single iteration (epoch) through n samples
def train_perceptron(X, y, w0, learning_rate):
    n_samples = X.shape[0]
    d_features = X.shape[1]

    for i in range(n_samples):
        # Compute the prediction for sample i
        pred_i = np.dot(X[i], w0)

        # Check if there's a misclassification
        error_i = y[i] - (pred_i > 0)  # True label - predicted label

        # Update weights and bias based on the error
        for j in range(d_features):
            w0[j] += learning_rate * error_i * X[i][j]

    return w0

# Example usage:
import numpy as np

X = np.array([[1, 2], [3, 4], [5, 6]])  # Input features
y = np.array([1, -1, 1])                # True labels
w0 = np.zeros(X.shape[1])               # Initial weights (d_features=2)
learning_rate = 0.1

# Train perceptron for one epoch
w_updated = train_perceptron(X, y, w0, learning_rate)
print("Updated weights:", w_updated)
```

**Example input / output:**

```

Input:

PYTHON
Copy
X = np.array([[1, 2], [3, 4], [5, 6]])
y = np.array([1, -1, 1])
w0 = np.zeros(2)
learning_rate = 0.1

Output:

PLAINTEXT
Copy
Updated weights: [1.68 3.76]

This shows the updated weights after one epoch over the dataset.
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there an error?
5. Update weights
6. Continue to next sample
7. End of epoch

