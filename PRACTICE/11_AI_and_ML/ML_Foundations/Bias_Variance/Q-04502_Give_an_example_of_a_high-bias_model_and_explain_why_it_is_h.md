---
type: "practice"
difficulty: "Basic"
---

# Give an example of a high-bias model and explain why it is high bias.

**Difficulty:** Basic · **Levels:** Fresher · [↑ Bias Variance Fundamentals](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Bias_Variance_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In machine learning, the bias-variance tradeoff refers to the balance between a model's ability to accurately fit the training data (low bias) and its generalization on unseen data (low variance). A high-bias model is one that underfits the training data, making it unable to capture important patterns or relationships. Let's explore an example of a high-bias model using simple linear regression.

#### Triaging the Question

The question asks for an example of a high-bias model and why it has this characteristic. The key aspects we need to address are:

What is a high-bias model?
Why does it have this bias?

#### First-Principles Solution

A common example of a high-bias model is simple linear regression applied to data that inherently follows a more complex pattern, such as a polynomial relationship or a non-linear one. For instance, if we try to fit a straight line through data points that follow a quadratic curve, the model will systematically miss important patterns in the data.

Why It Is High Bias

A high-bias model is characterized by its inability to capture the underlying structure of the data. Here's why simple linear regression becomes a high-bias model in this context:

Underfitting: The model is too simplistic to represent the true relationship between features and target variables.
Simplicity vs Complexity: Linear models assume linearity, which may not hold for more complex datasets.

Let's consider a concrete example using Python code.

Code Example

We'll generate some synthetic data that follows a quadratic pattern and then fit it with a simple linear regression model.

PYTHON
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression

# Generate synthetic data
np.random.seed(0)
X = np.random.rand(100, 1) * 10
y = X**2 - 5 * X + np.random.randn(100, 1)

# Fit a simple linear regression model
lin_reg = LinearRegression()
lin_reg.fit(X, y)

# Predict using the model
X_pred = np.linspace(0, 10, 100)[:, np.newaxis]
y_pred = lin_reg.predict(X_pred)

# Plotting
plt.scatter(X, y, color='blue', label='Actual Data')
plt.plot(X_pred, y_pred, color='red', label='Predicted Model')
plt.title('High-Bias Linear Regression Example')
plt.xlabel('X')
plt.ylabel('Y')
plt.legend()
plt.show()
Analysis

In the plot:

The actual data (blue dots) follows a quadratic pattern.
The predicted model (red line) is a straight line, clearly underfitting the data.

This example illustrates that simple linear regression is unable to capture the underlying non-linear relationship in the data, leading to high bias and poor generalization on unseen data.

#### Closing Summary

A high-bias model like simple linear regression applied to data with complex patterns systematically fails to capture important relationships, resulting in underfitting. This leads to poor performance on both training and test data because the model is too simplistic for the complexity of the task at hand.

#### Common Mistakes/Follow-Ups
Incorrect Model Choice: Choosing a linear model when the relationship is non-linear.
Ignoring Feature Engineering: Not transforming features or adding polynomial terms to capture higher-order relationships.
Overfitting vs Underfitting: Distinguishing between models that overfit (low bias, high variance) and those that underfit (high bias).

By understanding these aspects, you can better choose the right model for your data and avoid common pitfalls.

