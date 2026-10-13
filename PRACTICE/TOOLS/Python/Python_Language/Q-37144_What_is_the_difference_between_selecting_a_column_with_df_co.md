---
type: "practice"
difficulty: "Basic"
---

# What is the difference between selecting a column with df['col'] and df[['col']], and what do you get back in each case?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pandas DataFrames](<../../../../TOOLS/Python/Python_Language/Pandas_DataFrames.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer your question about Python Pandas, when working with DataFrame objects df, using df['col'] vs df[['col']] involves different operations and returns different types of results.

TL;DR:

df['col']: Returns a Series.
df[['col']]: Returns a DataFrame.

Now, let's dive into the details. The primary difference lies in how these syntaxes handle the data returned by Pandas DataFrames.

#### Triaging the Question

The question is asking about two different ways to access columns in a Pandas DataFrame: df['col'] and df[['col']]. We need to understand what each syntax returns, why they are used differently, and the implications of these choices.

#### Assumptions
Assume you have a DataFrame named df.
You want to retrieve data from column 'col'.
Step-by-Step Solution

df['col']:

When you use single square brackets ([]) with a string, like df['col'], Pandas treats it as an attribute access.
This method returns the column 'col' as a Series object.

df[['col']]:

Using double square brackets ([][]) when accessing a DataFrame column is interpreted as returning a subset of columns from the DataFrame.
In this case, df[['col']] will return a DataFrame with a single column named 'col'.

#### Example
Code and Explanation
PYTHON
import pandas as pd

# Sample data
data = {'col1': [1, 2, 3], 'col2': ['A', 'B', 'C']}
df = pd.DataFrame(data)

print("Using df['col'] to get a Series:")
series_result = df['col1']
print(series_result)
# Output: 0    1
#         1    2
#         2    3
# Name: col1, dtype: int64

print("\nUsing df[['col']] to get a DataFrame:")
df_subset = df[['col1']]
print(df_subset)
#   col1
# 0    1
# 1    2
# 2    3

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, df['col'] returns a Series (a one-dimensional labeled array), while df[['col']] returns a DataFrame with a single column. The choice between these two methods depends on whether you need a Series or a DataFrame for further operations.

Some common mistakes:

Using df[['col']] when only a Series is needed, which might lead to extra overhead.
Confusing the type of data returned and misinterpreting it in further data manipulation steps.

If asked follow-ups, you could discuss performance implications, use cases where each form is preferred, or explore how these differ in more complex DataFrame operations. For instance, using df[['col1', 'col2']] might be more readable for getting multiple columns compared to df.loc[:, ['col1', 'col2']].

