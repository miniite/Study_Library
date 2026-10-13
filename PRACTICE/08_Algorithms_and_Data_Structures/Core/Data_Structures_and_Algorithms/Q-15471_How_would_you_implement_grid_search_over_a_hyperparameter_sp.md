---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement grid search over a hyperparameter space, and what is the combinatorial complexity of the search?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Complexity of Encodings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Complexity_of_Encodings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Grid search involves systematically searching through different combinations of hyperparameters to find the best configuration for a machine learning model. For a fresher or intermediate ML engineer, it's essential to understand both the implementation and the combinatorial complexity involved in this process.

#### Triage
What Is Grid Search?

Grid search is an exhaustive search technique where we manually specify a grid of hyperparameter values and then train models using every combination of these parameters. This method ensures that no potential parameter setting is overlooked, but it can be computationally expensive if the number of possible combinations is large.

#### Assumptions
We have access to a machine learning model with multiple hyperparameters.
Each hyperparameter has a predefined set of values to test.
The goal is to find the best combination of these parameters that optimizes a specific metric (e.g., accuracy, loss).
Step-by-Step First-Principles Solution
1. Define Hyperparameters and Their Values

Let's assume we have two hyperparameters: learning_rate with values [0.001, 0.01, 0.1], and batch_size with values [32, 64, 128]. For this example, our grid will look like this:

PYTHON
learning_rates = [0.001, 0.01, 0.1]
batch_sizes = [32, 64, 128]
2. Generate All Possible Combinations

We can generate all possible combinations using nested loops or list comprehensions:

PYTHON
combinations = [(lr, bs) for lr in learning_rates for bs in batch_sizes]

This will result in the following combinations:

(0.001, 32)
(0.001, 64)
(0.001, 128)
(0.01, 32)
(0.01, 64)
(0.01, 128)
(0.1, 32)
(0.1, 64)
(0.1, 128)
3. Train the Model for Each Combination

For each combination, we train a model and evaluate its performance. Here’s an example using Python:

PYTHON
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression

# Sample dataset
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

best_params = None
best_score = float('-inf')

for lr, bs in combinations:
    model = LogisticRegression(learning_rate='constant', eta0=lr)
    model.fit(X_train, y_train)
    score = model.score(X_test, y_test)

    if score > best_score:
        best_params = {'learning_rate': lr, 'batch_size': bs}
        best_score = score

print(f"Best parameters: {best_params} with score: {best_score}")
4. Evaluate Combinatorial Complexity

The combinatorial complexity of the grid search is given by the product of the number of values each hyperparameter can take:

For learning_rate with 3 values.
For batch_size with 3 values.

Thus, the total number of combinations is (3 \times 3 = 9).

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, grid search involves generating all possible hyperparameter combinations and training models for each combination. The combinatorial complexity can quickly grow as the number of hyperparameters increases.

#### Common Mistakes

Not considering continuous hyperparameters (e.g., learning_rate should be continuously searched rather than just discretely).
Ignoring computational constraints, leading to infeasibly large search spaces.
Failing to properly validate or test models after training them on the entire dataset.

#### Follow-Ups

How would you modify this approach for continuous hyperparameters?
What are some strategies to reduce the complexity of grid search?
Can you explain how cross-validation fits into a grid search?

**Code:**

```python
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression

# Sample dataset
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

learning_rates = [0.001, 0.01, 0.1]
batch_sizes = [32, 64, 128]

combinations = [(lr, bs) for lr in learning_rates for bs in batch_sizes]

best_params = None
best_score = float('-inf')

for lr, bs in combinations:
    model = LogisticRegression(learning_rate='constant', eta0=lr)
    model.fit(X_train, y_train)
    score = model.score(X_test, y_test)

    if score > best_score:
        best_params = {'learning_rate': lr, 'batch_size': bs}
        best_score = score

print(f"Best parameters: {best_params} with score: {best_score}")
```

