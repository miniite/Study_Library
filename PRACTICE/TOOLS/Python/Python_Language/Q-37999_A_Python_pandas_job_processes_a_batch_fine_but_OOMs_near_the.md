---
type: "practice"
difficulty: "Deep"
---

# A Python pandas job processes a batch fine but OOMs near the end even though each row is small. What Python-specific memory pitfalls (accumulating lists, un-freed DataFrames, object dtype) would you check?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Memory Leaks in Batch Jobs](<../../../../TOOLS/Python/Python_Language/Memory_Leaks_in_Batch_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, a Python pandas job is functioning well in most cases but experiences an Out-of-Memory (OOM) error near the end of processing, even though each individual row is relatively small. This issue likely stems from specific memory management pitfalls common when using pandas and Python. Let's dive into the key Python-specific memory pitfalls that might be causing this problem.

#### Triaging the Question

The question is essentially asking to identify potential memory leaks or inefficient memory use in a pandas-based data processing job. The interviewer is probing your understanding of how pandas manages memory, common pitfalls associated with it, and your ability to diagnose such issues from first principles.

#### Assumptions
The OOM error occurs during the execution of the code.
Each row of data being processed is small but the dataset itself could be large enough to cause memory issues.
There are no hardware constraints or limitations (e.g., insufficient RAM).

#### First-Principles Solution
Accumulating Lists

One common pitfall is accumulating lists in Python, which can lead to significant memory consumption. Each time a list grows by appending new elements, the entire list has to be copied into a new location with additional space allocated for the appended items. This process is repeated every time an element is added, leading to quadratic growth of memory usage.

#### Solution
Replace accumulating lists with more efficient data structures or techniques such as using np.append in NumPy or adding elements directly to a pandas DataFrame if necessary.

PYTHON
# Bad: Accumulating list
data = []
for i in range(1000):
    data.append(i)

# Good: Using np.append (if applicable)
import numpy as np
arr = np.empty((1000,), dtype=np.int32)
for i in range(1000):
    arr[i] = i

# Better: Directly adding to DataFrame if appending rows is needed
df = pd.DataFrame(columns=['data'])
for i in range(1000):
    df.loc[len(df)] = [i]
Un-Freed DataFrames

Another issue can arise from not properly freeing memory used by pandas DataFrames. DataFrames hold on to memory until the object is explicitly deleted or garbage collected, which might not happen immediately if there are references still holding onto it.

#### Solution
Ensure that all unused DataFrame objects are explicitly deleted and their variables are set to None to allow for garbage collection.

PYTHON
import pandas as pd

# Example of freeing up memory by deleting unused DataFrames
df1 = pd.DataFrame({'data': range(1000)})
del df1  # Explicitly delete the DataFrame

# Alternatively, using context managers or resetting variables to None can also help
with pd.option_context('display.max_rows', None):
    df2 = pd.DataFrame({'data': range(5000)})
df2 = None
Object Dtypes

Pandas DataFrames with object dtype can consume a lot of memory, especially when storing string data. This is because strings are stored as Python objects, which have additional overhead compared to numeric types.

#### Solution
Convert columns from object dtype to more appropriate types if possible. For example, convert categorical or date-like strings into proper pandas dtypes like category, datetime64, etc., which can save memory and improve performance.

PYTHON
# Example of converting object dtype to category
df = pd.DataFrame({'data': ['a', 'b', 'c'] * 1000})
df['data'] = df['data'].astype('category')

# Avoiding unnecessary string storage by preprocessing data
def preprocess_data(series):
    return series.apply(lambda x: x.lower() if isinstance(x, str) else x)

df['processed_data'] = preprocess_data(df['raw_data'])

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, common memory pitfalls in a pandas-based job include accumulating lists that grow too large, not freeing up unused DataFrames and their references, and storing string data with the object dtype unnecessarily. To mitigate these issues, consider using more efficient data structures or converting dtypes where appropriate.

#### Common mistakes
to avoid:

Not recognizing the impact of memory-intensive operations.
Failing to manage DataFrame lifecycle properly.
Overlooking the overhead introduced by object dtype and unnecessary string processing.

#### Follow-up
questions might include:

How would you profile memory usage in a pandas job?
What strategies do you have for handling large datasets that don't fit into memory?

**Comparison:**

```
id	value
1	A
2	B
3	C
id	value
0	a
1	b
2	c
```

**Code:**

```python
# Example of accumulating lists vs. using np.append
import numpy as np

data = []
for i in range(1000):
    data.append(i)

arr = np.empty((1000,), dtype=np.int32)
for i in range(1000):
    arr[i] = i

# Example of freeing up memory by deleting unused DataFrames
df1 = pd.DataFrame({'data': range(1000)})
del df1  # Explicitly delete the DataFrame

# Example of converting object dtype to category
df = pd.DataFrame({'data': ['a', 'b', 'c'] * 1000})
df['data'] = df['data'].astype('category')

# Example of preprocessing data to avoid unnecessary string storage
def preprocess_data(series):
    return series.apply(lambda x: x.lower() if isinstance(x, str) else x)

df['processed_data'] = preprocess_data(df['raw_data'])
```

**Example input / output:**

```
Input

df = pd.DataFrame({"data": ['a', 'b', 'c'] * 1000})

Output after preprocessing

preprocessed_df = preprocess_data(df['data'])

print(preprocessed_df)

CODE
Copy
Output:

[0 1 2] dtype: int8
```

**Data:**

```json
{
  "data": [
    {"id": 1, "value": "A"},
    {"id": 2, "value": "B"},
    {"id": 3, "value": "C"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is DataFrame Large?
5. Check Accumulating Lists
6. Check Un-Freed DataFrames
7. Convert Object Dtypes toAppropriate Types
8. Profile Memory Usage
9. End

