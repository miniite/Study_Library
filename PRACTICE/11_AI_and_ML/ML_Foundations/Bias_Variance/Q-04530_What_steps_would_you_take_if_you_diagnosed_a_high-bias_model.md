---
type: "practice"
difficulty: "Intermediate"
---

# What steps would you take if you diagnosed a high-bias model?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Diagnosing Bias and Variance](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Diagnosing_Bias_and_Variance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When diagnosing a high-bias model, the key is to identify why the model's predictions are consistently poor, leading to underfitting. Typically, this means that the model is too simple or lacks enough complexity to capture the underlying patterns in the data. Here are the steps I would take:

Review and Understand the Data: Ensure the feature set is adequate and relevant. Explore whether any important features are missing.
Model Complexity Review: Assess if the current model's architecture, such as deep learning layers or decision tree depth, is insufficient for the problem.
Feature Engineering: Check if new or more complex features can improve performance.
Hyperparameter Tuning: Adjust hyperparameters like regularization strength, learning rate, and batch size to see if they can mitigate bias.
Model Validation: Use cross-validation techniques to ensure that the model's poor performance is not due to overfitting on a particular dataset split.

By following these steps, we aim to strike a balance between bias and variance and improve the model’s generalization ability.

**Comparison:**

```
Hyperparameters	Values
n_estimators	[10, 50, 100]
max_depth	[2, 4, 6]
```

**Code:**

```python
# Example of hyperparameter tuning using GridSearchCV in scikit-learn
from sklearn.model_selection import GridSearchCV

param_grid = {'n_estimators': [10, 50, 100], 'max_depth': [2, 4, 6]}
grid_search = GridSearchCV(estimator=RandomForestClassifier(), param_grid=param_grid, cv=5)
grid_search.fit(X_train, y_train)

best_params = grid_search.best_params_
print("Best parameters:", best_params)
```

**Example input / output:**

```text
# Example input and output for a simple dataset
X_train = [[0.1, 0.2], [0.4, 0.5]]
y_train = [0, 1]

# Running the hyperparameter tuning code
grid_search.fit(X_train, y_train)

best_params = grid_search.best_params_
print("Best parameters:", best_params)
```

**Data:**

```json
{
  "X_train": [
    [0.1, 0.2, 0.3],
    [0.4, 0.5, 0.6],
    ...
  ],
  "y_train": [0, 1, 0, ...]
}
```

**Flow:**

1. Review and Understand Data
2. Check Model Complexity
3. Add or Improve Features
4. Tune Hyperparameters
5. Validate Model
6. Repeat if Necessary

