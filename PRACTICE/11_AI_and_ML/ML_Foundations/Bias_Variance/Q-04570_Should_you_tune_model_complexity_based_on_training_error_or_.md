---
type: "practice"
difficulty: "Basic"
---

# Should you tune model complexity based on training error or validation error and why?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Diagnosing Bias and Variance](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Diagnosing_Bias_and_Variance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In machine learning, understanding the bias-variance tradeoff is crucial when tuning models. The key idea here is that a model should balance between fitting the training data too closely (high variance) and being too simplistic to capture important patterns (high bias). When it comes to choosing whether to tune based on training error or validation error, the answer lies in understanding these two concepts deeply.

We need to understand what each type of error represents. Training error measures how well a model fits the training data, while validation error provides insight into how well the model generalizes to unseen data. Tuning models based on the wrong metric can lead to suboptimal results.

Let's start by defining bias and variance in simpler terms:

Bias is the error due to overly simplistic assumptions in the learning algorithm. A high-bias model will oversimplify the problem, leading to underfitting.
Variance refers to the error due to a complex model that fits the training data too closely, including noise and outliers. High variance leads to overfitting.

Now, let's dive into why we should tune based on validation error:

Avoid Overfitting: Validation error helps us find the sweet spot where the model generalizes well to new data. If we optimize only for training error, the model might become too complex and start fitting noise in the training data (overfitting).

Generalization Performance: The goal of machine learning is usually not just to fit the training data but to perform well on unseen data. Validation error gives us a more accurate picture of how our model will behave in real-world scenarios.

Practical Example: Imagine you are building a spam filter for emails. If your model has low bias (it correctly identifies most spam), it's still not useful if its variance is high (it flags too many non-spam as spam). You need to balance these two aspects.

In practice, here’s how we can tune our models:

Initial Model Setup: Start with a simple model and gradually increase complexity.
Cross-Validation: Use k-fold cross-validation to get multiple validation errors for better reliability.
Learning Curves: Plot training error vs. validation error as you increase the complexity of your model. Typically, as complexity increases, training error decreases but validation error initially decreases then starts increasing (indicating overfitting).

Here’s a step-by-step reasoning:

Start Simple: Begin with a simple model and assess both training and validation errors.
Incremental Complexity: Gradually increase the complexity of your model.
Monitor Validation Error: Keep an eye on how the validation error changes as you add complexity. This will help you identify when the model starts to overfit.
Choose Optimal Model: Choose a model where the training and validation errors are close but not identical, indicating good generalization.

#### Common mistakes
to avoid:

Focusing Only on Training Error: Ignoring validation error can lead to models that perform well in training but poorly in real-world applications.
Overfitting Due to High Complexity: Increasing model complexity too much without validation monitoring can result in a model that captures noise instead of the underlying pattern.

In conclusion, tuning based on validation error is crucial for achieving good generalization and avoiding overfitting. By carefully balancing bias and variance, we ensure our models perform well not just during training but also when encountering new data.

**Comparison:**

| Complexity | Training Error | Validation Error |
|------------|----------------|------------------|
| 1          | 0.7            | 0.6              |
| 2          | 0.8            | 0.65             |
| 3          | 0.9            | 0.75             |
| 4          | 0.95           | 0.85             |
| 5          | 0.99           | 0.9              |

**Code:**

```python
# Example code snippet to illustrate the concept of learning curves
import numpy as np
from sklearn.model_selection import cross_val_score

def plot_learning_curves(model_complexity_range):
    train_errors = []
    validation_errors = []

    for complexity in model_complexity_range:
        model = create_model(complexity)
        # Simulate training and validation scores
        train_error = simulate_train_error(model)
        validation_error = cross_val_score(model, X, y, cv=5).mean()

        train_errors.append(train_error)
        validation_errors.append(validation_error)

    return train_errors, validation_errors

# Example function to create a model with a given complexity
def create_model(complexity):
    # Dummy model creation logic
    return f"Model_{complexity}"

# Simulate training error
def simulate_train_error(model):
    # Simulate some random data for demonstration purposes
    return np.random.rand()

train_errors, validation_errors = plot_learning_curves(range(1, 6))
```

**Example input / output:**

```text
| Complexity | Training Error | Validation Error |
|------------|----------------|------------------|
| 1          | 0.7            | 0.6              |
| 2          | 0.8            | 0.65             |
| 3          | 0.9            | 0.75             |
| 4          | 0.95           | 0.85             |
| 5          | 0.99           | 0.9              |

As complexity increases, training error decreases but validation error initially decreases then starts increasing.
```

**Data:**

```json
{
  "model_complexity_range": [1, 2, 3, 4, 5],
  "training_scores": [0.7, 0.8, 0.9, 0.95, 0.99],
  "validation_scores": [0.6, 0.65, 0.75, 0.85, 0.9]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Start
6. Is Model ComplexityIncreased?
7. Plot Learning Curves
8. Tune based on Validation Error
9. Check Training and ValidationErrors
10. If Training < Validation,Increase Complexity
11. Else, Optimal ComplexityFound
12. Is Model Overfitting?
13. Tuning Complete

