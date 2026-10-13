---
type: "practice"
difficulty: "Basic"
---

# Give an example of a high-variance model and explain why it is high variance.

**Difficulty:** Basic · **Levels:** Fresher · [↑ Ensembles and Variance Reduction](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Ensembles_and_Variance_Reduction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing machine learning, models can be classified based on their bias and variance. High-variance models tend to fit the training data too closely, often leading to poor generalization on new, unseen data. A common example of a high-variance model is a decision tree with deep levels or a polynomial regression with a very high degree.

Let's break down why this happens:

#### Overview
In simple terms, a high-variance model captures the noise in the training data rather than the underlying pattern.

Restating the Question: The question is asking us to identify an example of such a model and understand the reasons behind its high variance.

First-Principles Reasoning:

#### Example
A polynomial regression with a very high degree, say ( n = 10 ), in a dataset with only a few data points or simple trends.
Reasons for High Variance:
Overfitting: The model tries to fit every single noise point and thus fails to generalize well. Imagine having a small number of data points, and the high-degree polynomial fits each point exactly; it will not represent the underlying trend accurately when new data comes in.
Complexity: High complexity models (like deep decision trees) can easily capture complex relationships within the training set but are prone to overfitting. They may find intricate patterns that do not exist or are just noise, leading to poor performance on unseen data.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Summary: High-variance models like deep polynomial regressions with high degrees tend to fit the training data too closely due to their complexity, leading to overfitting.
Common Mistake: Often, developers might choose a model that is too complex for the dataset without considering the risk of overfitting. For example, using ( n = 10 ) in polynomial regression with just 5 data points can be overly ambitious and risky.

#### Follow-Up
Questions: Interviewers may probe deeper into how to mitigate this issue (e.g., regularization techniques, cross-validation), but for a fresher-level explanation, focusing on the concept of overfitting is sufficient.

**Comparison:**

```
Data Point	X	y
1	1	1.0
2	2	2.0
3	3	3.0
4	4	4.0
5	5	5.0
Polynomial Features (Degree=10) for X = [1, 2, 3, 4, 5]
[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
[2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048]
[3, 9, 27, 81, 243, 729, 2187, 6561, 19683, 59049, 177147]
[4, 16, 64, 256, 1024, 4096, 16384, 65536, 262144, 1048576, 4194304]
[5, 25, 125, 625, 3125, 15625, 78125, 390625, 1953125, 9765625, 48828125]
```

**Code:**

```python
import numpy as np
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression

# Example data
X = np.array([[1], [2], [3], [4], [5]])
y = np.array([1.0, 2.0, 3.0, 4.0, 5.0])

# Using a high-degree polynomial regression
poly = PolynomialFeatures(degree=10)
X_poly = poly.fit_transform(X)

model = LinearRegression()
model.fit(X_poly, y)
print("Model coefficients:", model.coef_)
```

**Example input / output:**

```text
# Example input and output for high-degree polynomial regression with cross-validation
from sklearn.model_selection import KFold

kfold = KFold(n_splits=5)
for train_index, test_index in kfold.split(X):
    X_train, X_test = X_poly[train_index], X_poly[test_index]
    y_train, y_test = y[train_index], y[test_index]

model.fit(X_train, y_train)
predictions = model.predict(X_test)

print("Predictions:", predictions)
```

**Data:**

```json
{
    "data": {
        "X": [[1], [2], [3], [4], [5]],
        "y": [1.0, 2.0, 3.0, 4.0, 5.0]
    },
    "poly_features": [
        [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
        [2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048],
        [3, 9, 27, 81, 243, 729, 2187, 6561, 19683, 59049, 177147],
        [4, 16, 64, 256, 1024, 4096, 16384, 65536, 262144, 1048576, 4194304],
        [5, 25, 125, 625, 3125, 15625, 78125, 390625, 1953125, 9765625, 48828125]
    ]
}
```

**Flow:**

1. Yes
2. No
3. High-degree PolynomialRegression
4. Are the data points few?
5. Overfitting likely
6. Higher complexity models arestill prone to overfitting if notcareful
7. Model captures noise insteadof underlying pattern
8. Might find intricate patternsthat do not exist or are justnoise
9. Mitigate with regularization,cross-validation
10. Mitigate with simpler modelsor feature selection

