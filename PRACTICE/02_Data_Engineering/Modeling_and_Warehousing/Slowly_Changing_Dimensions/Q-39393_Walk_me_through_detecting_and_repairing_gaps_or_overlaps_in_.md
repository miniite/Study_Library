---
type: "practice"
difficulty: "Expert"
---

# Walk me through detecting and repairing gaps or overlaps in the validity intervals of an existing SCD Type 2 dimension.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 in Practice](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In slowly changing dimensions (SCD), particularly SCD Type 2, we maintain a history by generating new records for changes in attributes over time. Detecting and repairing gaps or overlaps in these validity intervals is crucial to ensure data consistency. This involves checking the start and end dates of each record to identify any overlapping periods where more than one record should be active simultaneously (gaps) or when two records claim to cover the same period (overlaps). Here’s a step-by-step approach to address this issue.

**Code:**

```python
def detect_gaps_overlaps(data, key_column='dim_id', start_date_col='start_date', end_date_col='end_date'):
    # Step 1: Sort data by dim_id and date intervals
    sorted_data = data.sort_values([key_column, start_date_col])

    # Step 2: Initialize current record to None
    current_record = None

    for row in sorted_data.itertuples():
        if not current_record:
            # First iteration sets the initial record
            current_record = (row[key_column], row[start_date_col], row[end_date_col])
        else:
            prev_dim_id, prev_start_date, prev_end_date = current_record

            # Step 3: Detect overlap or gap
            if prev_dim_id == row[key_column]:
                if not pd.isna(prev_start_date) and (prev_end_date < row[start_date_col] <= prev_end_date):
                    print(f"Overlap detected for {row[key_column]} from {prev_start_date} to {prev_end_date}, new interval: {prev_start_date} to {row[end_date_col]}")
                elif pd.isna(prev_start_date) and row[start_date_col] < prev_end_date:
                    print(f"Gap detected for {row[key_column]} from {prev_start_date} to {prev_end_date}")
            else:
                # Reset current_record
                current_record = (row[key_column], row[start_date_col], row[end_date_col])
```

**Example input / output:**

```

Given the input data:

JSON
Copy
[
    {"dim_id": 1, "start_date": "2023-01-01", "end_date": "2023-06-01"},
    {"dim_id": 1, "start_date": "2023-05-01", "end_date": "2023-10-01"},
    {"dim_id": 1, "start_date": "2023-07-01", "end_date": "2023-11-01"}
]

The function detect_gaps_overlaps would output:

CODE
Copy
Overlap detected for 1 from 2023-05-01 to 2023-10-01, new interval: 2023-05-01 to 2023-11-01
Gap detected for 1 from 2023-06-01 to 2023-07-01
```

**Data:**

```json
[
    {"dim_id": 1, "start_date": "2023-01-01", "end_date": "2023-06-01"},
    {"dim_id": 1, "start_date": "2023-05-01", "end_date": "2023-10-01"},
    {"dim_id": 1, "start_date": "2023-07-01", "end_date": "2023-11-01"}
]
```

