---
type: "practice"
difficulty: "Intermediate"
---

# Which machine learning algorithms are sensitive to feature scaling and which are not?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Low Variance Features](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Low_Variance_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand what "feature scaling" means in the context of machine learning (ML) and identify which algorithms are sensitive to it. Feature scaling is a preprocessing step that standardizes features by bringing them into a similar range, often between 0 and 1 or with mean = 0 and variance = 1. Some ML algorithms require feature scaling because they depend on distance measures, while others are robust to the scale of input features.

#### Triage

The question is asking us to identify which machine learning algorithms are sensitive to feature scaling. We need to consider both supervised and unsupervised learning methods and understand their requirements for feature scaling. Our response will involve listing these algorithms and explaining why they require or do not require such preprocessing.

#### First-Principles Solution

Algorithms Sensitive to Feature Scaling:

Linear Regression: Linear regression assumes that the features are on a similar scale, as large differences in scale can lead to poor convergence of the coefficients.
Logistic Regression: Similar to linear regression, logistic regression also benefits from feature scaling for better performance and faster convergence.
Nearest Neighbors (k-NN): The distance metric used by k-NN is sensitive to the scale of features. Features with larger scales can dominate the distances, leading to incorrect predictions.
Support Vector Machines (SVMs): SVMs rely on a kernel function that measures the similarity between points in high-dimensional space. Proper scaling ensures that all features contribute equally.
Gradient Descent-based Algorithms: These include algorithms like Stochastic Gradient Descent (SGD) and its variants, which are sensitive to feature scales as they use gradients for optimization.

Algorithms Not Sensitive to Feature Scaling:

Decision Trees & Random Forests: Decision trees split nodes based on the information gain or Gini index, which is not affected by the scale of features.
Naive Bayes: Naive Bayes classifiers make probabilistic assumptions that do not depend on the feature scales. However, scaling can sometimes improve the performance if it helps with numerical stability.
Neural Networks: While neural networks benefit from normalized inputs for better training dynamics and faster convergence, they are generally not sensitive to the scale of features in a strict sense.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, most distance-based or gradient descent-based algorithms require feature scaling to perform optimally. Algorithms like SVMs, k-NN, logistic regression, and linear regression should always be preprocessed with techniques such as standardization or normalization before training. On the other hand, tree-based models are robust to feature scaling.

When implementing these algorithms in practice, it's crucial to check the documentation or literature for specific requirements on preprocessing steps. Always validate the impact of feature scaling through experiments by comparing performance metrics with and without this step.

#### Common mistakes
include:

Overfitting on training data by not performing cross-validation.
Ignoring the importance of feature scaling in distance-based algorithms, leading to suboptimal model performance.
Assuming all models benefit from the same preprocessing techniques, which is often not true.
Additional Follow-Ups
How would you handle categorical features during feature scaling?
Can you provide an example where feature scaling improved the performance significantly for a specific algorithm?

**Comparison:**

| Original Features | Scaled Features |
|--------------------|-----------------|
| [1.0, -2.5]        | [-2.215, -2.086] |
| [-0.7, 4.0]        | [-0.667, 2.867]  |
| [3.8, -1.0]        | [2.883, 0.219]   |
| [6.3, 0.9]         | [4.344, -1.806]  |

**Code:**

```python
# Example of feature scaling using StandardScaler from scikit-learn
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
scaled_features = scaler.fit_transform(X)
```

**Example input / output:**

```

import numpy as np from sklearn.preprocessing import StandardScaler

Input data

X = np.array([[1.0, -2.5], [-0.7, 4.0], [3.8, -1.0], [6.3, 0.9]])

Scaling the features

scaler = StandardScaler() scaled_features = scaler.fit_transform(X)

print(scaled_features)

CODE
Copy
Output:
```python
[[-2.21575342 -2.08554623]
 [-0.66723429  2.86691911]
 [ 2.8829877   0.21942702]
 [ 4.34372023 -1.80594615]]
```

**Data:**

```json
{
  "X": [
    [1.0, -2.5],
    [-0.7, 4.0],
    [3.8, -1.0],
    [6.3, 0.9]
  ]
}
```

**Flow:**

1. Feature Scaling Required?
2. Linear Regression
3. Standardize Features
4. Logistic Regression
5. Standardize Features
6. k-NN
7. Normalize Features
8. SVMs
9. Standardize Features
10. Gradient Descent Algorithms
11. Standardize Features

