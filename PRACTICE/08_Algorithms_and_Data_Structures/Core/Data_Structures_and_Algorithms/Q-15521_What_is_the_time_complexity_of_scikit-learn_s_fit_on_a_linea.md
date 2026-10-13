---
type: "practice"
difficulty: "Basic"
---

# What is the time complexity of scikit-learn's fit on a linear model versus predict, and why is predict typically much cheaper?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Complexity of Encodings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Complexity_of_Encodings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this discussion, we will explore the time complexities associated with fitting and predicting using a linear model in scikit-learn. Specifically, I'll explain why the fit operation on a linear model is typically much more computationally intensive than the predict operation.

TL;DR

The time complexity of scikit-learn's fit for a linear model is (O(n \cdot d^2 + n \cdot d)), where (n) is the number of samples and (d) is the number of features. In contrast, the predict operation has a time complexity of (O(n \cdot d)). The difference in computational cost arises from the need for matrix operations during fitting compared to simply performing vector-matrix multiplications during prediction.

#### Triaging the Question

When asked about the time complexities of fit and predict methods in scikit-learn's linear models, the interviewer is probing your understanding of fundamental concepts in data structures and algorithms. Key assumptions include knowledge of linear algebra and basic machine learning principles. The main idea is to articulate why the fitting process is more resource-intensive than making predictions.

Step-by-Step Solution
1. Fitting a Linear Model (fit)

Fitting a linear model involves solving for the coefficients that minimize the loss function, typically using methods like ordinary least squares (OLS). Here’s what happens under the hood:

Matrix Operations: To fit the model, we need to solve the normal equation: ( \mathbf{w} = (\mathbf{X}^T \mathbf{X})^{-1} \mathbf{X}^T \mathbf{y} ).

Here, (\mathbf{X}) is the design matrix of size (n \times d), and (\mathbf{y}) is the target vector of length (n).

Time Complexity: The dominant operations are:

Calculating (\mathbf{X}^T \mathbf{X}): This requires (O(d^2 n)) multiplications.
Inverting the matrix ((\mathbf{X}^T \mathbf{X})^{-1}): For a dense matrix, this step is (O(d^3)).
Calculating (\mathbf{w} = (\mathbf{X}^T \mathbf{X})^{-1} \mathbf{X}^T \mathbf{y}): This requires additional multiplications and additions.

Thus, the overall time complexity for fitting can be approximated as (O(d^2 n + d n)).

2. Predicting with a Linear Model (predict)

Predicting involves using the trained model to make predictions on new data:

Vector-Matrix Multiplication: For each sample in the test set, we perform vector-matrix multiplication (\mathbf{w}^T \mathbf{x}), where (\mathbf{w}) is a (d)-dimensional vector and (\mathbf{x}) is a feature vector of length (d).

Time Complexity: For each sample, the operation is straightforward: (O(d)). If there are (n) samples in the test set, then the total complexity becomes (O(n d)).

#### Closing Summary

To summarize, the fit method's time complexity for a linear model in scikit-learn involves heavy matrix operations, particularly inversion, which leads to a higher computational cost. In contrast, the predict method relies on simpler vector-matrix multiplications, making it significantly cheaper.

#### Common Mistakes
and Follow-Ups
Mistake: Ignoring the impact of matrix inversion in the fit process.

#### Follow-Up
Discussing other methods like gradient descent or using sparse data structures to mitigate the complexity issues during fitting.
Additional Insight: Mentioning that for very large datasets, approximations or incremental learning techniques might be used to reduce computational overhead.

**Code:**

```python
# Example of fitting and predicting in scikit-learn
from sklearn.linear_model import LinearRegression

# Sample data (X: features, y: target)
X = [[1], [2], [3], [4]]
y = [2, 3, 5, 7]

# Create a linear regression model
model = LinearRegression()

# Fit the model to the data
model.fit(X, y)

# Predict using the fitted model
predictions = model.predict([[5], [6]])

print(predictions)
```

**Example input / output:**

```text
X = [[1], [2], [3], [4]]
y = [2, 3, 5, 7]

model.fit(X, y)

predictions = model.predict([[5], [6]])
print(predictions) # Output: [8.0, 9.0]
```

