---
type: "practice"
difficulty: "Intermediate"
---

# How would you use GridSearchCV in scikit-learn to tune hyperparameters, and what does the cv parameter control?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Normalization with NumPy](<../../../../TOOLS/Python/Python_Language/Vector_Normalization_with_NumPy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

GridSearchCV is a powerful tool in scikit-learn used to find the best hyperparameters by performing an exhaustive search over specified parameter values. For a fresher or intermediate ML engineer, it's important to understand how GridSearchCV works and what each of its parameters controls. The cv parameter specifically controls the cross-validation strategy that is used to evaluate each combination of hyperparameters.

Let’s break down how to use GridSearchCV, starting from first principles:

TL;DR
What you're doing: Use GridSearchCV to find the best hyperparameters.
Key idea: GridSearchCV exhaustively searches through a grid of hyperparameter values and uses cross-validation to evaluate each combination.

#### Triage
Understanding GridSearchCV

The question is asking how to use GridSearchCV in scikit-learn to tune hyperparameters, specifically focusing on the role of the cv parameter. The cv parameter controls the number and type of folds used for cross-validation during the search process.

#### Assumptions
You have a dataset and a machine learning model.
You want to find the best combination of hyperparameters using grid search.
You need to understand how cross-validation works in this context.
Step-by-Step Solution

Import Libraries:

PYTHON
from sklearn.model_selection import GridSearchCV, train_test_split
from sklearn.datasets import load_iris
from sklearn.svm import SVC

Load or Create a Dataset: For demonstration purposes, we'll use the Iris dataset.

PYTHON
iris = load_iris()
X, y = iris.data, iris.target

Define the Model and Hyperparameters to Search: We’ll use an SVM model with different values of C (regularization parameter) and gamma.

PYTHON
param_grid = {'C': [0.1, 1, 10, 100], 'gamma': [1, 0.1, 0.01, 0.001]}
model = SVC()

Initialize GridSearchCV: Here, cv is set to 5, meaning a 5-fold cross-validation will be used.

PYTHON
grid_search = GridSearchCV(estimator=model, param_grid=param_grid, cv=5)

Fit the Model: Fit the model on the training data.

PYTHON
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
grid_search.fit(X_train, y_train)

Evaluate and Interpret Results: After fitting, you can access the best parameters and score.

PYTHON
print("Best Parameters: ", grid_search.best_params_)
print("Best Score: ", grid_search.best_score_)
Full Code Example
PYTHON
from sklearn.model_selection import GridSearchCV, train_test_split
from sklearn.datasets import load_iris
from sklearn.svm import SVC

# Load or create a dataset
iris = load_iris()
X, y = iris.data, iris.target

# Define the model and hyperparameters to search
param_grid = {'C': [0.1, 1, 10, 100], 'gamma': [1, 0.1, 0.01, 0.001]}
model = SVC()

# Initialize GridSearchCV with 5-fold cross-validation
grid_search = GridSearchCV(estimator=model, param_grid=param_grid, cv=5)

# Fit the model on training data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
grid_search.fit(X_train, y_train)

# Evaluate and interpret results
print("Best Parameters: ", grid_search.best_params_)
print("Best Score: ", grid_search.best_score_)

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, GridSearchCV is a robust tool for hyperparameter tuning by performing an exhaustive search over a specified parameter space. The cv parameter controls the cross-validation strategy used to evaluate each combination of parameters. A common mistake is setting cv too high or too low, which can either increase runtime unnecessarily or result in overly optimistic performance estimates.

#### Follow-Up
Questions:

How does changing the value of cv affect the computation time and model evaluation?
What are some alternatives to GridSearchCV for hyperparameter tuning?

**Code:**

```python
from sklearn.model_selection import GridSearchCV, train_test_split
from sklearn.datasets import load_iris
from sklearn.svm import SVC

# Load or create a dataset
iris = load_iris()
X, y = iris.data, iris.target

# Define the model and hyperparameters to search
param_grid = {'C': [0.1, 1, 10, 100], 'gamma': [1, 0.1, 0.01, 0.001]}
model = SVC()

# Initialize GridSearchCV with 5-fold cross-validation
grid_search = GridSearchCV(estimator=model, param_grid=param_grid, cv=5)

# Fit the model on training data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
grid_search.fit(X_train, y_train)

# Evaluate and interpret results
print("Best Parameters: ", grid_search.best_params_)
print("Best Score: ", grid_search.best_score_)
```

**Example input / output:**

```text
# Example input
X_train, X_test, y_train, y_test = train_test_split(iris.data, iris.target, test_size=0.2, random_state=42)

# Example output
Best Parameters: {'C': 100, 'gamma': 0.001}
Best Score: 0.9666666666666667
```

**Data:**

```json
{
  "param_grid": {
    "C": [0.1, 1, 10, 100],
    "gamma": [1, 0.1, 0.01, 0.001]
  }
}
```

