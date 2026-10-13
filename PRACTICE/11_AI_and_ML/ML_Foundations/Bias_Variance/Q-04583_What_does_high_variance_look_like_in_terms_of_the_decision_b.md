---
type: "practice"
difficulty: "Basic"
---

# What does high variance look like in terms of the decision boundary of a classifier?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Diagnosing Bias and Variance](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Diagnosing_Bias_and_Variance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

High variance in a classifier's decision boundary often results in a model that is overly complex and fits too closely to the training data, leading to poor generalization on unseen data. In simpler terms, it means the model has become too "flexible" and captures noise instead of the underlying pattern.

To understand this concept further, let's break down what variance represents in machine learning:

Definition: Variance measures how much the decision boundary changes with different training datasets.
Symptoms in Decision Boundaries: In a classifier, high variance can be observed as an overly intricate and wiggly decision boundary that closely follows noise or outliers in the data.

Now, let's reason through this from first principles:

Step-by-Step Reasoning

Understanding Bias-Variance Decomposition:

The error of a model can be decomposed into three components: bias, variance, and irreducible error.
For simplicity, we'll focus on the bias and variance parts.

Bias: Represents how far off the predictions are from the true value on average. High bias means underfitting, where the model is too simple to capture the data's underlying pattern.

Variance: Measures the model’s sensitivity to small fluctuations in the training set. High variance indicates overfitting, meaning the model captures noise and details that do not generalize well.

Visualizing High Variance

Imagine we have a dataset with two classes: positive (red) and negative (blue). In a high-variance scenario:

Training Data: Consider an example where both classes have some overlap.
Decision Boundary: A classifier like a decision tree or a highly complex neural network might create a very intricate boundary to fit the training data closely. This can result in many small, jagged segments that capture noise and outliers.

#### Example
Decision Tree

Simple Dataset:

CODE
X1 | X2 | Class
---------------
 0 | 0  | Red
 0 | 1  | Blue
 1 | 0  | Red
 1 | 1  | Blue

High Variance Decision Tree: A decision tree might split the data based on more complex rules, like:

Split X1 at 0.4.
Further splits based on X2 values close to boundaries between classes.

Visual Representation: The decision boundary would look very detailed and jagged, with many small segments that follow the noise in the training data closely.

#### Closing Summary

High variance manifests as a decision boundary that is overly complex and captures noise or outliers in the training data, leading to poor performance on new, unseen data. This can be seen visually as a highly intricate and wiggly boundary that closely follows the training set's irregularities.

#### Common Mistakes/Follow-Ups
Mistake: Focusing only on simple models (low variance) without considering bias.

#### Follow-Up
Discuss how to balance bias and variance using techniques like cross-validation, regularization, or ensemble methods.

**Comparison:**

```
Feature	Value
Bias	Low
Variance	High
Model Complexity	High
```

**Code:**

```python
# Example code for a decision tree with high variance
from sklearn.tree import DecisionTreeClassifier
import numpy as np

# Dummy data generation
X = np.array([[0, 0], [1, 1], [0.4, 0.5], [0.6, 0.8]])
y = np.array([0, 1, 0, 1])

# Training a decision tree with high variance
dt = DecisionTreeClassifier(max_depth=3)
dt.fit(X, y)

print(dt.predict([[0.5, 0.5]]))  # Might return either class due to complexity
```

**Example input / output:**

```text
Decision Tree Prediction:
Input: [[0.5, 0.5]]
Output: [0] or [1] depending on the noise captured in training data.
```

**Data:**

```json
{
  "X": [[0, 0], [1, 1], [0.4, 0.5], [0.6, 0.8]],
  "y": [0, 1, 0, 1]
}
```

**Flow:**

1. Yes
2. No
3. High Variance
4. Does the model fit noise?
5. Complex decision boundary
6. Simple and generalizable
7. Overfitting, poorgeneralization
8. Underfitting, high bias

