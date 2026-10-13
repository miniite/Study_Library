---
type: "practice"
difficulty: "Intermediate"
---

# The AI generates code that imputes missing values with the global mean including test rows. Why is that leakage, and how do you fix it?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Reviewing AI Code for Correctness Bugs](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Correctness_Bugs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When an AI-assisted tool like Copilot or GitHub's Cursor generates code that imputes missing values with the global mean (or any aggregate statistic) including test rows, this introduces data leakage. Data leakage occurs when information from outside the training set is used in the model building process, leading to overly optimistic performance metrics during validation.

#### Triaging the Question

The question is asking why using a global mean for imputing missing values in the test set is problematic and how we can address it without losing valuable information. The interviewer is probing your understanding of data leakage and its impact on model performance evaluation.

#### Assumptions
Global Mean Imputation: This means replacing all missing values with the overall mean of the dataset.
Training vs. Test Set: Typically, the training set is used to train the model, while the test set is kept separate for evaluating the model's true performance.
Data Leakage: Using information from the test set during training can lead to biased and overly optimistic results.
Step-by-Step Solution:
1. Understanding Data Leakage

Data leakage occurs when data from outside the training dataset is used in the process of building a model, leading to models that perform well on historical data but poorly on new, unseen data. In the context of imputing missing values, using the global mean (or any aggregate statistic) calculated from the entire dataset—including the test set—can introduce leakage.

2. Example Scenario

Let's say we have a small dataset with some missing values in our features:

MARKDOWN
| feature1 | feature2 | target |
|----------|----------|--------|
| 3        | 5        | 1      |
| 6        |          | 0      |
| 9        | 7        | 1      |
|          | 8        | 0      |

If we impute the missing values using the global mean of the entire dataset (including both training and test sets), our imputed data would look like:

MARKDOWN
| feature1 | feature2 | target |
|----------|----------|--------|
| 3        | 5.67     | 1      |
| 6        | 5.67     | 0      |
| 9        | 7        | 1      |
| 8.33     | 8        | 0      |

This global mean is calculated from the entire dataset, which includes both training and test data points.

3. Impact on Model Evaluation

Using this imputed value during model evaluation will give us an overly optimistic performance metric because the model has effectively "seen" the test data through the imputation process. This makes our validation metrics unreliable as they no longer reflect how well the model would perform in real-world scenarios where it only sees new, unseen data.

4. Fixing Data Leakage

To avoid this issue, we need to ensure that the imputation is performed on a per-fold basis during cross-validation or separately for training and test sets:

Cross-Validation:

Perform k-fold cross-validation.
Impute missing values using only the training data within each fold before making predictions on the validation set.
This way, information from the validation set is not used in the imputation process.

Hold-out Validation:

Split the dataset into a training and test set once at the beginning.
Impute missing values using only the training data.
Use the trained model to make predictions on the test set without any further imputation, as the test set should remain untouched for evaluation.

#### Closing Summary

Data leakage from global mean imputation can significantly bias model performance metrics. To avoid this, always ensure that imputation is done using only the training data and that the test set remains completely separate during both the training and evaluation processes.

#### Common Mistakes/Follow-Ups
Mistake: Using aggregate statistics (e.g., global mean) calculated from the entire dataset for imputing missing values.

#### Follow-Up
How would you handle categorical features with missing values? Would using mode be better than mean?
Further Discussion: Discuss scenarios where cross-validation might not suffice and other methods like iterative imputation are necessary.

**Comparison:**

| Fold | Train Indices | Val Indices |
|------|---------------|-------------|
| 1    | [0, 1]        | [2, 3]      |
| 2    | [1, 2]        | [0, 3]      |
| 3    | [0, 3]        | [1, 2]      |
| 4    | [1, 2]        | [0, 3]      |
| 5    | [0, 3]        | [1, 2]      |

**Code:**

```python
# Example of performing imputation separately for each fold in k-fold cross-validation

from sklearn.model_selection import KFold
import numpy as np

def impute_missing_values(X_train, X_val):
    # Impute missing values using the training data only
    mean_imputer = SimpleImputer(strategy='mean')
    X_train_imp = mean_imputer.fit_transform(X_train)

    # Use the trained imputer to transform validation set
    X_val_imp = mean_imputer.transform(X_val)
    return X_train_imp, X_val_imp

# Sample usage in k-fold cross-validation
kf = KFold(n_splits=5, shuffle=True, random_state=42)
for train_idx, val_idx in kf.split(X):
    X_train, y_train = X[train_idx], y[train_idx]
    X_val, y_val = X[val_idx], y[val_idx]

    X_train_imp, X_val_imp = impute_missing_values(X_train, X_val)
```

**Example input / output:**

```text
Input (Before Imputation):
feature1 | feature2 | target
3        | 5        | 1
6        |          | 0
9        | 7        | 1
          | 8        | 0

Imputed Data:
feature1 | feature2 | target
3        | 5.67     | 1
6        | 5.67     | 0
9        | 7        | 1
8.33      | 8        | 0
```

**Data:**

```yaml
# Example dataset schema for demonstration
data:
  - feature1: [3, 6, 9]
    feature2: [5, null, 7]
  - target: [1, 0, 1]

imputed_data:
  - feature1: [3, 6, 9]
    feature2: [5.67, 5.67, 7]
  - target: [1, 0, 1]
```

**Flow:**

1. Train Model
2. Impute Missing Values
3. Use Mean Imputation
4. Split into Train and Test
5. Apply Imputation Separatelyfor Each Fold
6. Cross-Validate
7. Evaluate Model Performance

