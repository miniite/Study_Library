---
type: "practice"
difficulty: "Basic"
---

# What does groupby do in pandas, and how would you compute the average of a numeric column for each category?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pandas DataFrames](<../../../../TOOLS/Python/Python_Language/Pandas_DataFrames.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Groupby is a powerful tool in pandas that allows us to perform operations on subsets of data defined by labels or conditions. In this context, we will focus on its usage to compute the average of a numeric column for each category.

#### Triaging the Question

The question asks two main things:

What does groupby do in pandas?
How would you compute the average of a numeric column for each category using groupby?

#### Assumptions

We are working with a DataFrame that contains at least one categorical column and one numeric column.
The numeric column is intended to be averaged.
Step-by-Step Solution
1. What Does Groupby Do in Pandas?

Groupby is used to split the data into groups based on some criteria, apply a function to each group independently, and combine the results back together. In essence, it allows you to operate on subsets of your DataFrame defined by certain conditions.

2. Computing the Average Using groupby

To compute the average (mean) of a numeric column for each category in pandas, follow these steps:

PYTHON
import pandas as pd

# Sample Data: A simple example with categories and values.
data = {
    'Category': ['A', 'B', 'C', 'A', 'B', 'C'],
    'Value': [10, 20, 30, 40, 50, 60]
}

# Creating the DataFrame
df = pd.DataFrame(data)

# Using groupby to split the data into groups based on 'Category'
grouped_data = df.groupby('Category')

# Applying the mean function to each group in 'Value' column
average_values = grouped_data['Value'].mean()

print(average_values)

In this example, groupby('Category') creates a group for each unique value in the Category column. The mean() function is then applied to the Value column within these groups.

Full Solution Code
PYTHON
import pandas as pd

# Sample Data: A simple example with categories and values.
data = {
    'Category': ['A', 'B', 'C', 'A', 'B', 'C'],
    'Value': [10, 20, 30, 40, 50, 60]
}

# Creating the DataFrame
df = pd.DataFrame(data)

# Using groupby to split the data into groups based on 'Category'
grouped_data = df.groupby('Category')

# Applying the mean function to each group in 'Value' column
average_values = grouped_data['Value'].mean()

print(average_values)

#### Closing Summary
and Common Mistakes/Follow-Ups

In this example, we computed the average of a numeric column (Value) for each category (Category). The key is understanding that groupby splits the DataFrame into groups defined by categories, and then you can apply any function to these groups.

#### Common Mistakes

Incorrectly specifying the columns to group by or apply functions.
Not understanding how to handle non-numeric data types when grouping.

#### Follow-Ups

How would you compute the sum instead of the mean?
If there are multiple numeric columns, how would you perform these operations simultaneously?

**Comparison:**

| Category | Value |
|----------|-------|
| A        | 10    |
| B        | 20    |
| C        | 30    |
| A        | 40    |
| B        | 50    |
| C        | 60    |

**Code:**

```python
import pandas as pd

# Sample Data: A simple example with categories and values.
data = {
    'Category': ['A', 'B', 'C', 'A', 'B', 'C'],
    'Value': [10, 20, 30, 40, 50, 60]
}

# Creating the DataFrame
df = pd.DataFrame(data)

# Using groupby to split the data into groups based on 'Category'
grouped_data = df.groupby('Category')

# Applying the mean function to each group in 'Value' column
average_values = grouped_data['Value'].mean()

print(average_values)
```

**Example input / output:**

```

import pandas as pd

Sample Data: A simple example with categories and values.

data = { 'Category': ['A', 'B', 'C', 'A', 'B', 'C'], 'Value': [10, 20, 30, 40, 50, 60] }

Creating the DataFrame

df = pd.DataFrame(data)

Using groupby to split the data into groups based on 'Category'

grouped_data = df.groupby('Category')

Applying the mean function to each group in 'Value' column

average_values = grouped_data['Value'].mean()

print(average_values)

CODE
Copy
Output:

A 25.0 B 35.0 C 45.0 dtype: float64
```

**Data:**

```json
{
    "data": {
        "Category": ["A", "B", "C", "A", "B", "C"],
        "Value": [10, 20, 30, 40, 50, 60]
    }
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Define DataFrame
6. Data Contains Categories andValues?
7. Create DataFrame
8. Group Data by Category usinggroupby()
9. Apply Mean Function toNumeric Column?
10. Print Average Values
11. G
12. Output: Averages per Category
13. Misunderstood GroupbyUsage

