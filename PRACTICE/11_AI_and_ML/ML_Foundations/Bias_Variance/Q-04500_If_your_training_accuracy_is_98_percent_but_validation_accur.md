---
type: "practice"
difficulty: "Intermediate"
---

# If your training accuracy is 98 percent but validation accuracy is 70 percent what is likely happening?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Diagnosing Bias and Variance](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Diagnosing_Bias_and_Variance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are facing a common issue in machine learning known as high bias and low variance. The training accuracy of 98% indicates that the model fits the training data very well, but the validation accuracy of 70% suggests poor generalization to new, unseen data. This discrepancy points towards overfitting.

To break down the question:

What is being probed: The interviewer is assessing your understanding of how models perform on different datasets (training vs. validation) and the concept of bias-variance tradeoff.
Key assumptions: Assume a binary classification problem, where accuracy metrics are relevant for performance evaluation.

Let's dive into the first-principles solution:

Understanding Overfitting:

A model with high training accuracy but low validation accuracy is likely overfitted to the training data. This means the model has learned noise and details specific to the training set that do not generalize well to new data.

Bias-Variance Decomposition:

Bias refers to the error introduced by approximating a real-world problem, which may be complex, with a simplified model. High bias can cause underfitting where the model is too simple and cannot capture relevant patterns in the data.
Variance measures how much the prediction changes when we train on different subsets of the training data. Low variance indicates that small fluctuations in the training set will not significantly affect the predictions, which is desirable but not always achievable.

Analyzing the Scenario:

High bias: The model might be too simple to capture the underlying patterns in the validation data.
High variance: The model might be too complex and is capturing noise or overfitting the training data.

Possible Causes of Overfitting:

Insufficient regularization: If the model’s complexity is not controlled, it can fit the noise in the training data.
Not enough data: Training on a small dataset relative to the model's capacity can lead to overfitting.
Model architecture: A complex model with many parameters can easily learn the noise.

Mitigation Strategies:

Use cross-validation to ensure the model generalizes well across different subsets of the training data.
Apply regularization techniques like L1, L2, or dropout to control the complexity and prevent overfitting.
Collect more data if possible, which can help the model generalize better.
Simplify the model architecture by reducing the number of layers or neurons.

#### Closing Summary
The primary issue here is that the model has overfitted to the training data, leading to high variance and low validation accuracy. To improve performance, focus on regularization and ensure the model generalizes well across different datasets.

**Comparison:**

```
Bias	Variance	Overfitting
Low	High	Yes
```

**Code:**

```python
# Example of implementing L2 regularization in a simple linear regression using scikit-learn
from sklearn.linear_model import Ridge

# Initialize the Ridge model with a high alpha (lambda) value to enforce stronger regularization
ridge_model = Ridge(alpha=1.0)

# Train the model on the training data
ridge_model.fit(X_train, y_train)

# Predict on validation data and evaluate performance
validation_predictions = ridge_model.predict(X_val)
```

**Example input / output:**

```text
Training Data:
X_train = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
y_train = [0.8, 0.9, 0.7]

Validation Data:
X_val = [[10, 11, 12], [13, 14, 15], [16, 17, 18]]
y_val = [0.6, 0.5, 0.4]

Model: Ridge Regression (alpha=1.0)

Training:
- Model trains on X_train and y_train

Validation:
- Predictions: [0.82, 0.93, 0.72]
- Validation accuracy: ~75%
```

**Data:**

```json
{
  "X_train": [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
  ],
  "y_train": [0.8, 0.9, 0.7],
  "X_val": [
    [10, 11, 12],
    [13, 14, 15],
    [16, 17, 18]
  ],
  "y_val": [0.6, 0.5, 0.4]
}
```

**Flow:**

1. :
2. Yes
3. No
4. Yes
5. No
6. Overfitting
7. Is the model too complex?
8. Apply regularizationtechniques
9. Not enough data?
10. Collect more data
11. Simplify model architecture

