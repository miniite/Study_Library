---
type: "practice"
difficulty: "Expert"
---

# How would you canary a new version of a batch pipeline by running it in shadow (writing to a side table) and comparing outputs before promoting it to write the real table?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ ML Model Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To canary a new version of a batch pipeline, we need to run the updated code on a small subset of data or in shadow mode (writing to a side table) and compare its outputs with those from the current stable version before promoting it to write directly into the main production table. This process helps us ensure that the changes introduced are valid, do not introduce bugs, and align with expected behavior.

The key idea here is to implement a controlled deployment strategy where we can verify the new pipeline's output without affecting live data until we are confident in its stability and correctness.

**Comparison:**

| Column | New Pipeline Output | Stable Pipeline Output |
|--------|---------------------|-----------------------|
| A      | AB                  | A                     |
| C      | CD                  | C                     |

**Code:**

```python
# Example Python code snippet for canary deployment

import pandas as pd

def read_data(file_path):
    return pd.read_csv(file_path)

def process_data(df, new_pipeline_code=True):
    if new_pipeline_code:
        # Apply the new pipeline logic here
        df = df.assign(new_column=df['column1'] + df['column2'])
    else:
        # Apply the old stable pipeline logic here
        df = df.assign(new_column=df['column1'].str.upper())
    return df

def write_to_production(df, table_name):
    # Write to the production database/table
    pass

def write_to_side_table(df, side_table_name):
    # Write to a side or staging database/table for canary testing
    pass

# Load data
data = read_data('path/to/data.csv')

# Process with new pipeline (canary)
new_output = process_data(data, new_pipeline_code=True)

# Process with old stable pipeline
stable_output = process_data(data, new_pipeline_code=False)

# Write the output of both pipelines to respective tables
write_to_side_table(new_output, 'side_table_new_version')
write_to_production(stable_output, 'production_table_stable_version')

# Compare outputs
comparison_results = compare_tables('side_table_new_version', 'production_table_stable_version')
```

**Example input / output:**

```

Input (Sample Data)

JSON
Copy
{
  "data": [
    {"column1": "A", "column2": "B"},
    {"column1": "C", "column2": "D"}
  ]
}

Output (Processed by New Pipeline)

JSON
Copy
{
  "output_new_pipeline": [
    {"column1": "A", "column2": "B", "new_column": "AB"},
    {"column1": "C", "column2": "D", "new_column": "CD"}
  ]
}

Output (Processed by Stable Pipeline)

JSON
Copy
{
  "output_stable_pipeline": [
    {"column1": "A", "column2": "B", "new_column": "A"},
    {"column1": "C", "column2": "D", "new_column": "C"}
  ]
}

Comparison Results

JSON
Copy
{
  "comparison_results": [
    true, // Matches
    false // Does not match due to different transformation logic
  ]
}
```

**Data:**

```json
{
  "data": [
    {"column1": "A", "column2": "B"},
    {"column1": "C", "column2": "D"}
  ]
}
```

**Flow:**

1. Read Data
2. Process with New Pipeline
3. Write to Side Table
4. Compare Outputs
5. Promote to Production?
6. Yes, Promote to Production
7. No, Debug and Retry

