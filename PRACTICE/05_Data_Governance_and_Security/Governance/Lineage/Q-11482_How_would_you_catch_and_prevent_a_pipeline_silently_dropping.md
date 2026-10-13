---
type: "practice"
difficulty: "Deep"
---

# How would you catch and prevent a pipeline silently dropping rows so lineage shows a link but data completeness is wrong?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lineage for Dynamic SQL](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Lineage_for_Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of catching and preventing a pipeline from silently dropping rows while maintaining accurate lineage, we need to establish robust mechanisms that ensure both data integrity and thorough tracking of data transformations. This involves leveraging advanced monitoring tools, implementing comprehensive validation checks, and establishing detailed metadata policies.

The key idea is to design a system where every row's transformation or deletion is logged and verified, ensuring that the lineage accurately reflects the state of the data at each stage of processing. By doing so, we can detect anomalies early and prevent data completeness issues from arising.

**Comparison:**

| Before Processing | After Processing |
|--------------------|------------------|
| ID: 1, Value: 10   | ID: 1, Value: 10  |
| ID: 2, Value: 20   | ID: 2, Value: 20  |
| ID: 3, Value: 30   |                  |

**Code:**

```python
# Example Python function for checking row count consistency during data processing
def check_row_count_inconsistency(input_df, output_df):
    input_count = input_df.shape[0]
    output_count = output_df.shape[0]

    if input_count != output_count:
        logging.error(f"Row count mismatch: Input {input_count}, Output {output_count}")
        return False
    else:
        return True

# Example usage in a data processing pipeline step
def process_data(input_path, output_path):
    df_input = pd.read_csv(input_path)

    # Process the data here
    processed_df = transform(df_input)

    # Check row count before writing to disk
    if not check_row_count_inconsistency(df_input, processed_df):
        raise ValueError("Row count mismatch detected")

    processed_df.to_csv(output_path, index=False)
```

**Example input / output:**

```

Input:

JSON
Copy
[
  {"id": 1, "value": 10},
  {"id": 2, "value": 20},
  {"id": 3, "value": 30}
]

Output:

JSON
Copy
{
  "before_processing": [
    {"id": 1, "value": 10},
    {"id": 2, "value": 20},
    {"id": 3, "value": 30}
  ],
  "after_processing": [
    {"id": 1, "value": 10},
    {"id": 2, "value": 20}  # Row 3 was dropped
  ]
}
```

**Data:**

```json
{
  "data": [
    {"id": 1, "value": 10},
    {"id": 2, "value": 20},
    {"id": 3, "value": 30}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Ingest
4. Transformation Step
5. Check Row Count
6. Database Write
7. Failure Alert
8. Lineage Update

