---
type: "practice"
difficulty: "Basic"
---

# How do you use train_test_split from scikit-learn, and why do you pass random_state and stratify?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Normalization with NumPy](<../../../../TOOLS/Python/Python_Language/Vector_Normalization_with_NumPy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To begin with, the train_test_split function in scikit-learn is a fundamental tool used to divide datasets into training and testing sets, which are crucial for evaluating machine learning models. The core concept revolves around setting aside a portion of your dataset to test the model’s performance independently from how it was trained.

When you use train_test_split, there are two key parameters that need attention: random_state and stratify. Understanding these will help in ensuring that the split is both reproducible and representative of the original data distribution.

**Comparison:**

| Split Data | Train Set (X_train) | Target (y_train) |
|------------|---------------------|------------------|
| X          | [1, 3, 5, 7, 9]     | [0, 0, 0, 0, 0]  |
| y          |                      |
| Split Data | Test Set (X_test) | Target (y_test) |
|------------|-------------------|-----------------|
| X          | [2, 4, 6, 8, 10]  | [1, 1, 1, 1, 1] |
| y          |                   |

**Code:**

```python
from sklearn.model_selection import train_test_split

# Example usage with random_state and stratify
X = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]  # Feature data (for illustration)
y = [0, 1, 0, 1, 0, 1, 0, 1, 0, 1]   # Target variable

# Splitting the dataset
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)
```

**Example input / output:**

```text
# Input
X_train, X_test, y_train, y_test = train_test_split(feature_data, target_variable, test_size=0.2, random_state=42, stratify=target_variable)

# Output
X_train: [1, 3, 5, 7, 9]
y_train: [0, 0, 0, 0, 0]

X_test: [2, 4, 6, 8, 10]
y_test: [1, 1, 1, 1, 1]
```

**Data:**

```python
# Example data schema
feature_data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]  # X
target_variable = [0, 1, 0, 1, 0, 1, 0, 1, 0, 1]  # y
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Split Data
6. Random State?
7. Reproducible Split
8. Stratify?
9. Stratified Split
10. Non-Stratified Split

