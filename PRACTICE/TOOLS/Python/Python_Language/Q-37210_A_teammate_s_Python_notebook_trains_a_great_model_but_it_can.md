---
type: "practice"
difficulty: "Intermediate"
---

# A teammate's Python notebook trains a great model but it can't be productionized. How would you refactor exploratory notebook code into a testable, parameterized pipeline module?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Offline Evaluation Harnesses](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/Offline_Evaluation_Harnesses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To refactor an exploratory machine learning (ML) notebook into a testable and parameterized pipeline module in Python, we need to follow several key steps. First, let’s summarize what we aim to achieve: converting ad-hoc code into modular, reusable components that are well-structured, easy to maintain, and robust. We will refactor the existing code by breaking it down into functions and classes, introducing parameters where necessary, and ensuring thorough testing.

The goal is not just to make the code more readable but also to ensure that each component can be tested independently, making the overall system more reliable. This process requires understanding the data flow, identifying dependencies, and defining clear interfaces between different parts of the pipeline.

**Comparison:**

```
Function	Purpose
load_data	Load the dataset from a file
preprocess_features	Preprocess and split features
train_model	Train the machine learning model
evaluate_model	Evaluate the model performance
```

**Code:**

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

def load_data(filepath):
    return pd.read_csv(filepath)

def preprocess_features(df, target_column='target'):
    X = df.drop(columns=[target_column])
    y = df[target_column]
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)
    return X_train_scaled, y_train, X_test

def train_model(X_train, y_train):
    model = LinearRegression()
    model.fit(X_train, y_train)
    return model

def evaluate_model(model, X_test, y_test):
    predictions = model.predict(X_test)
    mse = mean_squared_error(y_test, predictions)
    return mse
```

**Example input / output:**

```text
# Example input data and parameters
file_path = 'train.csv'
X_train, y_train, X_test = load_data(file_path)
model = train_model(X_train, y_train)
mse = evaluate_model(model, X_test)
print(f"Mean Squared Error: {mse}")
```

**Data:**

```json
{
  "data": [
    {
      "filename": "train.csv",
      "columns": ["feature1", "feature2", "target"]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Load Data
4. Preprocess Features
5. Split Data?
6. Train Model
7. Evaluate Model
8. Return Trained Model
9. Calculate MSE
10. Store Model
11. Output MSE

