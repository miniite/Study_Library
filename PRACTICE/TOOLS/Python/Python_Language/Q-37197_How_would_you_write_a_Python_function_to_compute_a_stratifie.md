---
type: "practice"
difficulty: "Deep"
---

# How would you write a Python function to compute a stratified time-based split for a time-series model so you never train on the future, and why does naive shuffling break temporal models?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Drift-triggered Retraining](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Drift-triggered_Retraining.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of splitting a dataset for a time-series model in a way that prevents training on future data, we need to ensure that any date in the training set is strictly before any date in the validation or test sets. This can be achieved by using stratified sampling based on timestamps while respecting the temporal order.

The key idea is to partition the data into periods and then split each period independently while ensuring that all historical data comes first, followed by future data. Here’s how we can implement this in Python:

Understand What's Being Probed:

The interviewer is looking for a deep understanding of time-series data handling.
They want to ensure you understand the importance of maintaining temporal integrity during splits.
You need to demonstrate knowledge of stratified sampling and how it can be adapted for time-based splitting.

#### Assumptions

The dataset contains timestamps in some sort of chronological order.
We will use a fixed split ratio, such as 80% train, 10% validation, and 10% test.

#### First-Principles Solution

PYTHON
import pandas as pd
from sklearn.model_selection import KFold

def stratified_time_split(data, target_col, time_col, shuffle=False):
    """
    Perform a stratified time-based split on a DataFrame.

    Parameters:
        data (pd.DataFrame): The dataset containing the target and timestamp columns.
        target_col (str): Name of the column to be used for stratification.
        time_col (str): Name of the column representing timestamps.
        shuffle (bool): Whether or not to shuffle the rows before splitting.

    Returns:
        train_df (pd.DataFrame): The training set.
        val_df (pd.DataFrame): The validation set.
        test_df (pd.DataFrame): The test set.
    """

    # Ensure time_col is in datetime format
    data[time_col] = pd.to_datetime(data[time_col])
    data.sort_values(by=time_col, inplace=True)

    # Calculate the split points based on cumulative proportions
    cum_proportions = data.groupby(target_col).apply(lambda x: (x.index.values - 0) / len(x))
    train_split_idx = int(len(cum_proportions) * 0.8)  # 80% training

    # Split the data into stratified time-based partitions
    train_df = cum_proportions.iloc[:train_split_idx]
    val_test_cumulative = cum_proportions.iloc[train_split_idx:]
    val_split_idx = int(len(val_test_cumulative) * 0.5)  # 50% for validation, 50% for test

    val_df = val_test_cumulative.iloc[:val_split_idx]
    test_df = val_test_cumulative.iloc[val_split_idx:]

    return train_df.index, val_df.index, test_df.index

#### Closing Summary
and Common Mistakes/Follow-Ups:

Summary: The function stratified_time_split takes a DataFrame with timestamps and a target column and splits the data into training, validation, and test sets while respecting temporal order.

#### Common Mistakes

Not sorting by timestamp before splitting.
Ignoring the stratification aspect, leading to unbalanced classes in the time-based split.
Using naive shuffling (shuffle=True), which breaks the temporal integrity of the data.

#### Follow-Ups

How would you handle different split ratios?
What about dealing with seasonality or external factors affecting the model?
How do you ensure that the function is efficient for large datasets?

**Comparison:**

```
Train Indices	Validation Indices	Test Indices
[0, 1, 2, ... ]	[80, 81, 82, ... ]	[90, 91, 92, ... ]
```

**Code:**

```python
import pandas as pd

# Example data
data = pd.DataFrame({
    'timestamp': pd.date_range(start='2023-01-01', periods=100, freq='D'),
    'target': [i % 5 for i in range(100)]
})
data['timestamp'] = pd.to_datetime(data['timestamp'])
```

**Example input / output:**

```text
# Applying the function to example data
train_idx, val_idx, test_idx = stratified_time_split(data, 'target', 'timestamp')

print("Train indices:", train_idx)
print("Validation indices:", val_idx)
print("Test indices:", test_idx)

train_data = data.iloc[train_idx]
val_data = data.iloc[val_idx]
test_data = data.iloc[test_idx]

print(train_data.head(), "\n")
print(val_data.head(), "\n")
print(test_data.head())
```

**Data:**

```python
# Example data creation
data = pd.DataFrame({
    'timestamp': pd.date_range(start='2023-01-01', periods=100, freq='D'),
    'target': [i % 5 for i in range(100)]
})
data['timestamp'] = pd.to_datetime(data['timestamp'])
```

**Flow:**

1. Load Data
2. Sort by Timestamp
3. Calculate CumulativeProportions
4. Split into Training, Validation,and Test
5. Return Indices for Each Set

