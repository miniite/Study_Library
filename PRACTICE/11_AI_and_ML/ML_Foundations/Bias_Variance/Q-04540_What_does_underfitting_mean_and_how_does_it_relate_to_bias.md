---
type: "practice"
difficulty: "Basic"
---

# What does underfitting mean and how does it relate to bias?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Diagnosing Bias and Variance](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Diagnosing_Bias_and_Variance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Underfitting occurs when a machine learning model is too simple to capture the underlying patterns in the data, leading to poor performance on both training and unseen data. It's closely related to the concept of bias, which refers to the error introduced by approximating a real-world problem, which might be complex, with a simplified model.

To understand underfitting and its relation to bias, let’s first look at how models are evaluated using the bias-variance tradeoff framework.

**Code:**

```python
# A simple function to demonstrate overfitting and underfitting in terms of prediction error
def evaluate_model(model, X_train, y_train, X_test, y_test):
    train_error = model.score(X_train, y_train)
    test_error = model.score(X_test, y_test)
    return train_error, test_error

# Example models for illustration (these would be replaced with actual ML models in practice)
linear_model = LinearRegression()
polynomial_model = PolynomialFeatures(degree=10)

X_train, X_test, y_train, y_test = load_data()  # Assume this function loads the data
```

**Example input / output:**

```text
# Example of evaluating a simple linear model and a high-degree polynomial model
train_error_linear, test_error_linear = evaluate_model(linear_model, X_train, y_train, X_test, y_test)
print(f"Linear Model Train Error: {train_error_linear}, Test Error: {test_error_linear}")

polynomial_model.fit(X_train, y_train)
train_error_poly, test_error_poly = evaluate_model(polynomial_model, X_train, y_train, X_test, y_test)
print(f"Polynomial Model (High Degree) Train Error: {train_error_poly}, Test Error: {test_error_poly}")
```

**Data:**

```json
{
  "example_data": {
    "features": [1, 2, 3, 4, 5],
    "labels": [0.8, 1.9, 3, 4.1, 5.2]
  }
}
```

