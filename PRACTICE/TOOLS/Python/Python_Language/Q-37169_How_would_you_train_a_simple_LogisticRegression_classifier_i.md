---
type: "practice"
difficulty: "Basic"
---

# How would you train a simple LogisticRegression classifier in scikit-learn and get predictions on a test set in Python?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Normalization with NumPy](<../../../../TOOLS/Python/Python_Language/Vector_Normalization_with_NumPy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To train a simple Logistic Regression classifier using scikit-learn and make predictions, we first need to understand that logistic regression is a fundamental binary classification algorithm used in machine learning. This model predicts the probability of an instance belonging to one of two classes by fitting data to a logistic function.

#### Triaging the Question

The question asks us to train a simple Logistic Regression classifier using scikit-learn and make predictions on a test set. The interviewer is probing our understanding of basic machine learning workflows, including data preprocessing, model training, and making predictions. We will assume that the dataset is already preprocessed and split into training and testing sets.

#### First-Principles Solution
Import Necessary Libraries:
Import LogisticRegression from sklearn.linear_model.
Import other necessary utilities like train_test_split for splitting data.
Load and Preprocess Data: Assume the dataset is in a CSV file named data.csv with features in columns 1-3 and labels in column 4.
Split Data into Training and Testing Sets:
Use train_test_split to split the data.
Train the Model:
Initialize a Logistic Regression model.
Fit the model on the training data.
Make Predictions:
Use the trained model to predict labels for the test set.
Detailed Steps

Import Libraries:

PYTHON
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression

Load Data:

PYTHON
# Load data from CSV file
df = pd.read_csv('data.csv')

# Separate features and labels
X = df.iloc[:, :3]  # Features: first three columns
y = df.iloc[:, 3]   # Labels: fourth column

Split Data:

PYTHON
# Split data into training and testing sets (80% train, 20% test)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

Train the Model:

PYTHON
# Initialize and fit the logistic regression model
model = LogisticRegression()
model.fit(X_train, y_train)

Make Predictions:

PYTHON
# Make predictions on the test set
y_pred = model.predict(X_test)

# Print predictions
print("Predictions:", y_pred)

#### Closing Summary
and Common Mistakes/Follow-Ups

This is a basic example of how to train a logistic regression model using scikit-learn. Common mistakes include not properly splitting the data, making typos in library imports or function calls, or forgetting to fit the model. Additionally, it's important to consider feature scaling if features are on different scales.

Code Example
PYTHON
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression

# Load data from CSV file
df = pd.read_csv('data.csv')

# Separate features and labels
X = df.iloc[:, :3]  # Features: first three columns
y = df.iloc[:, 3]   # Labels: fourth column

# Split data into training and testing sets (80% train, 20% test)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Initialize and fit the logistic regression model
model = LogisticRegression()
model.fit(X_train, y_train)

# Make predictions on the test set
y_pred = model.predict(X_test)
print("Predictions:", y_pred)
Self-Check

Correctness + completeness confirmed.

**Comparison:**

| Feature | Description |
|---------|-------------|
| feature1 | First input feature |
| feature2 | Second input feature |
| feature3 | Third input feature |
| label    | Target class (binary) |

**Code:**

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression

# Load data from CSV file
df = pd.read_csv('data.csv')

# Separate features and labels
X = df.iloc[:, :3]  # Features: first three columns
y = df.iloc[:, 3]   # Labels: fourth column

# Split data into training and testing sets (80% train, 20% test)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Initialize and fit the logistic regression model
model = LogisticRegression()
model.fit(X_train, y_train)

# Make predictions on the test set
y_pred = model.predict(X_test)
print("Predictions:", y_pred)
```

**Example input / output:**

```text
# Example input data (partial)
X = [[1.0, 2.0, 3.0], [4.0, 5.0, 6.0], [7.0, 8.0, 9.0]]
y = [0, 1, 0]

# Example output
Predictions: [0 1 0]
```

**Data:**

```
CSV
Copy
# data.csv
feature1, feature2, feature3, label
1.0, 2.0, 3.0, 0
4.0, 5.0, 6.0, 1
7.0, 8.0, 9.0, 0
...
```

**Flow:**

1. Load Data
2. Separate Features and Labels
3. Split Data into Training andTesting Sets
4. Initialize Logistic RegressionModel
5. Fit the Model on Train Data
6. Predict Test Set Labels
7. Print Predictions

