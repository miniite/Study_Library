---
type: "practice"
difficulty: "Intermediate"
---

# What is a data freshness or timeliness check and why does a stale dataset cause problems?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Detecting Unit Changes](<../../../../02_Data_Engineering/Data_Quality/Validation/Detecting_Unit_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Data freshness, also known as timeliness, refers to how current your data is at any given point in time. It's crucial because outdated data can lead to incorrect business decisions and lost opportunities. Imagine a company using sales data from last year to make this year’s marketing strategy—those insights would likely be irrelevant or even misleading.

To understand why stale datasets cause problems, we need to break down the concept of freshness and its importance in different contexts.

**Comparison:**

| ID | Date       | Revenue |
|----|------------|---------|
| 1  | 2023-01-01 | 1500    |
| 2  | 2023-01-05 | 2000    |

**Code:**

```python
# Example Python code for checking data freshness (simplified)
import pandas as pd

def check_data_freshness(df, last_update_col):
    current_time = pd.Timestamp.now()
    latest_update = df[last_update_col].max()

    if latest_update < current_time:
        print(f"Data is stale. Latest update: {latest_update}, Current time: {current_time}")
    else:
        print("Data is fresh.")
```

**Example input / output:**

```
Example input and output for the check_data_freshness function
Input

data = pd.DataFrame.from_dict({ "id": [1, 2], "date": ["2023-01-01", "2023-01-05"], "revenue": [1500, 2000] }) last_update_col = 'date'

Output

check_data_freshness(data, last_update_col)

CODE
Copy
Output:

Data is fresh.
```

**Data:**

```json
{
  "sales_data": [
    {"id": 1, "date": "2023-01-01", "revenue": 1500},
    {"id": 2, "date": "2023-01-05", "revenue": 2000}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Freshness Check
4. Is the latest update daterecent?
5. Fresh data
6. Stale data, trigger alert
7. Proceed with analysis anddecisions
8. Investigate source of delay,update system

