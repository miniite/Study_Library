---
type: "practice"
difficulty: "Deep"
---

# Your training pipeline's label join drops 15% of rows silently. How would you instrument the Python join to surface row-count deltas and detect a broken key or a schema change upstream?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Reconciliation Scripts](<../../../../TOOLS/Python/Python_Language/Data_Reconciliation_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where your training pipeline’s label join operation drops 15% of rows silently, we need to instrument the Python code to log row-count deltas and detect any potential issues such as a broken key or schema changes upstream. This involves understanding the current behavior, identifying the root cause, and implementing robust logging mechanisms.

#### Triage

The question is asking how to debug and monitor a data join operation that silently drops rows by more than 15%, ensuring we can trace the problem back to its source. The core challenge here is dealing with missing or misaligned keys during the join process. We need to log relevant information to pinpoint exactly where these issues arise.

Step-by-Step Solution

Understand the Current Behavior:

Identify the data sources and their schemas.
Determine the key fields being used for the join operation.

Instrument the Code:

Add logging statements at various stages of the join process to capture row counts before and after the join.
Implement checks to validate the keys involved in the join.

Implement Robust Logging Mechanisms:

Log row counts for each dataset before joining.
Log the count of matched rows during the join operation.
Log any missing or mismatched key pairs that cause rows to be dropped.

Detect and Handle Schema Changes:

Implement schema validation mechanisms to check if the data format has changed upstream.
Use a version control system for datasets to ensure consistency over time.

Debugging and Validation:

Run the pipeline with logging enabled and review the logs to identify the root cause of row drops.
Validate that the logging covers all critical points in the join process, including key validation and schema checks.
Detailed Solution

Let’s walk through a detailed implementation:

Identify Data Sources and Key Fields:

PYTHON
# Example data sources
df1 = pd.read_csv('data1.csv')
df2 = pd.read_csv('data2.csv')

# Identify key fields for join
key_field_1 = 'id'
key_field_2 = 'customer_id'

Add Logging Statements:

PYTHON
import logging

logging.basicConfig(level=logging.INFO)

def log_row_counts(df, prefix):
    """Log the row count of a DataFrame."""
    count = len(df)
    logging.info(f"{prefix} row count: {count}")

# Log initial row counts for each data frame
log_row_counts(df1, 'Before join - df1')
log_row_counts(df2, 'Before join - df2')

# Perform the join operation
joined_df = pd.merge(df1, df2, left_on=key_field_1, right_on=key_field_2)

# Log the count of matched rows during the join
log_row_counts(joined_df, 'Joined row count')

Implement Key Validation:

PYTHON
def validate_keys(row):
    """Validate if both keys are present and non-null."""
    return not pd.isnull(row[key_field_1]) and not pd.isnull(row[key_field_2])

# Filter out rows where the key validation fails
valid_rows = joined_df[joined_df.apply(validate_keys, axis=1)]

# Log the count of valid rows after key validation
log_row_counts(valid_rows, 'Valid row count')

# Further process only the valid rows
processed_df = valid_rows.drop(columns=['key_field_1', 'key_field_2']).reset_index(drop=True)

Detect and Handle Schema Changes:

PYTHON
def check_schema_changes(df, reference_df):
    """Check if there are any schema changes in the DataFrame."""
    # Example: Check if all columns from the reference df are present
    missing_columns = set(reference_df.columns) - set(df.columns)
    if missing_columns:
        logging.warning(f"Schema change detected: Missing columns {missing_columns}")

# Reference data (from a previous known good state)
ref_df1 = pd.read_csv('ref_data1.csv')
ref_df2 = pd.read_csv('ref_data2.csv')

check_schema_changes(df1, ref_df1)
check_schema_changes(df2, ref_df2)

if not valid_rows.empty:
    # Log a warning for schema changes in the joined data
    check_schema_changes(processed_df, pd.merge(ref_df1, ref_df2, on=key_field_2))

#### Closing Summary
and Common Mistakes/Follow-Ups

By instrumenting the join operation with logging statements and key validation checks, we can identify and address row drops caused by missing or misaligned keys. Additionally, schema validation ensures that any upstream changes are detected early in the pipeline.

#### Common Mistakes

Not logging all critical stages of the join process.
Ignoring missing or non-null validations for key fields.
Failing to check if downstream processing uses valid rows only.

#### Follow-Ups

Implement automated schema validation across multiple datasets and versions.
Integrate these checks into a CI/CD pipeline to catch issues early in development cycles.
Regularly update reference schemas based on historical data to ensure consistency over time.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Data Join Logging and Validation
    dateFormat YYYY-MM-DD
    section Initial Setup
    df1_import:     "read_csv('data1.csv')"
    df2_import:     "read_csv('data2.csv')"
    ref_df1_import: "read_csv('ref_data1.csv')"
    ref_df2_import: "read_csv('ref_data2.csv')"

    section Logging Initial Row Counts
    log_row_counts_df1: "log_row_counts(df1, 'Before join - df1')"
    log_row_counts_df2: "log_row_counts(df2, 'Before join - df2')"

    section Join Operation
    joined_df_merge: "pd.merge(df1, df2, left_on=key_field_1, right_on=key_field_2)"
    log_joined_rows: "log_row_counts(joined_df, 'Joined row count')"

    section Key Validation
    validate_keys: "validate_keys(row)"
    valid_rows_filter: "valid_rows = joined_df[joined_df.apply(validate_keys, axis=1)]"
    log_valid_rows: "log_row_counts(valid_rows, 'Valid row count')"

    section Process Valid Rows
    processed_df_drop: "processed_df = valid_rows.drop(columns=[key_field_1, key_field_2]).reset_index(drop=True)"

    section Schema Validation
    schema_changes_check: "check_schema_changes(df1, ref_df1)"
    schema_changes_check_processed: "check_schema_changes(processed_df, pd.merge(ref_df1, ref_df2, on=key_field_2))"
```

**Code:**

```python
import logging

logging.basicConfig(level=logging.INFO)

def log_row_counts(df, prefix):
    count = len(df)
    logging.info(f"{prefix} row count: {count}")

df1 = pd.read_csv('data1.csv')
df2 = pd.read_csv('data2.csv')

key_field_1 = 'id'
key_field_2 = 'customer_id'

log_row_counts(df1, 'Before join - df1')
log_row_counts(df2, 'Before join - df2')

joined_df = pd.merge(df1, df2, left_on=key_field_1, right_on=key_field_2)
log_row_counts(joined_df, 'Joined row count')

def validate_keys(row):
    return not pd.isnull(row[key_field_1]) and not pd.isnull(row[key_field_2])

valid_rows = joined_df[joined_df.apply(validate_keys, axis=1)]
log_row_counts(valid_rows, 'Valid row count')

processed_df = valid_rows.drop(columns=[key_field_1, key_field_2]).reset_index(drop=True)

def check_schema_changes(df, reference_df):
    missing_columns = set(reference_df.columns) - set(df.columns)
    if missing_columns:
        logging.warning(f"Schema change detected: Missing columns {missing_columns}")

ref_df1 = pd.read_csv('ref_data1.csv')
ref_df2 = pd.read_csv('ref_data2.csv')

check_schema_changes(df1, ref_df1)
check_schema_changes(df2, ref_df2)

if not valid_rows.empty:
    check_schema_changes(processed_df, pd.merge(ref_df1, ref_df2, on=key_field_2))
```

**Example input / output:**

| Before join - df1 row count | Joined row count | Valid row count |
|-----------------------------|------------------|----------------|
| 3                           | 2                | 2              |

**Data:**

```json
{
  "data1.csv": {
    "columns": ["id", "value"],
    "rows": [
      {"id": 1, "value": "A"},
      {"id": 2, "value": "B"},
      {"id": 3, "value": "C"}
    ]
  },
  "data2.csv": {
    "columns": ["customer_id", "value"],
    "rows": [
      {"customer_id": 1, "value": "X"},
      {"customer_id": 4, "value": "Y"},
      {"customer_id": 3, "value": "Z"}
    ]
  },
  "ref_data1.csv": {
    "columns": ["id", "value"],
    "rows": [
      {"id": 1, "value": "A"},
      {"id": 2, "value": "B"},
      {"id": 3, "value": "C"}
    ]
  },
  "ref_data2.csv": {
    "columns": ["customer_id", "value"],
    "rows": [
      {"customer_id": 1, "value": "X"},
      {"customer_id": 4, "value": "Y"},
      {"customer_id": 3, "value": "Z"}
    ]
  }
}
```

**Flow:**

1. Yes
2. Read Data Files
3. Log Initial Row Counts
4. Join Operation?
5. Perform Join
6. Validate Keys
7. Filter Valid Rows
8. Log Valid Row Count
9. Process Valid Rows
10. Check Schema Changes

