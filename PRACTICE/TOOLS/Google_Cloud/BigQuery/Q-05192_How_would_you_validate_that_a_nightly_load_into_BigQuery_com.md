---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate that a nightly load into BigQuery completed correctly?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ BigQuery Loading and Table Design](<../../../../TOOLS/Google_Cloud/BigQuery/BigQuery_Loading_and_Table_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a nightly load into BigQuery completes correctly, we need to design and implement a robust validation process. This involves checking both the data integrity and the completeness of the loaded dataset. We can break this down step-by-step starting from basic checks to more advanced ones.

#### Triage

The core question here is about verifying the success of an ETL (Extract, Transform, Load) pipeline that uses BigQuery as the target database. The interviewer is probing your understanding of data validation techniques, error handling, and the overall robustness of a data loading process in BigQuery.

#### First-Principles Solution
Source Data Validation: Ensure the source dataset meets certain criteria (e.g., schema correctness, data quality).
Data Load Verification: Confirm that the data was successfully loaded into BigQuery.
Post-Load Checks: Validate the integrity and completeness of the loaded data.
Step 1: Source Data Validation

Before loading, we should validate the source dataset to ensure it is ready for ingestion. This includes checking:

Schema consistency with expected column names and types.
Missing values or nulls in critical columns.
Data format correctness (e.g., dates, numbers).
PYTHON
# Example Python validation code
import pandas as pd

def validate_source_data(source_df):
    # Check schema: Ensure all required columns are present
    required_columns = ['user_id', 'transaction_date', 'amount']
    missing_cols = [col for col in required_columns if col not in source_df.columns]

    if missing_cols:
        raise ValueError(f"Missing columns: {missing_cols}")

    # Check data format (e.g., date parsing)
    try:
        pd.to_datetime(source_df['transaction_date'], errors='raise')
    except ValueError as e:
        print(f"Date validation failed: {e}")
        return False

    return True
Step 2: Data Load Verification

Once the source data is validated, we proceed with loading it into BigQuery. Post-loading, we should confirm that the data was indeed loaded and verify its integrity.

PYTHON
# Example Python script for loading data to BigQuery
from google.cloud import bigquery

def load_data_to_bigquery(source_df):
    client = bigquery.Client()

    dataset_id = "my_dataset"
    table_id = f"{dataset_id}.transactions"

    job_config = bigquery.LoadJobConfig(
        schema=[
            bigquery.SchemaField("user_id", "STRING"),
            bigquery.SchemaField("transaction_date", "DATE"),
            bigquery.SchemaField("amount", "NUMERIC")
        ],
        skip_leading_rows=1,
        write_disposition="WRITE_TRUNCATE"
    )

    job = client.load_table_from_dataframe(source_df, table_id, job_config=job_config)
    job.result()  # Waits for the load job to complete

    return True
Step 3: Post-Load Checks

After loading, we perform several checks to ensure data integrity and completeness:

Row Count Verification: Confirm that the number of rows loaded matches expectations.
Data Distribution Check: Ensure data is distributed across partitions or clusters as expected.
Null Values Check: Verify no critical columns have null values.
PYTHON
# Example Python validation code for post-load checks
def validate_loaded_data():
    client = bigquery.Client()

    query = """
    SELECT COUNT(*) AS total_rows, COUNT(user_id) AS user_count,
           COUNTIF(amount IS NULL) AS null_amount_count
    FROM `my_dataset.transactions`
    """

    query_job = client.query(query)
    results = query_job.result().to_dataframe()

    expected_row_count = 100000  # Example expected value

    if results['total_rows'].values[0] != expected_row_count:
        print(f"Row count mismatch: Expected {expected_row_count}, got {results['total_rows'].values[0]}")

    if results['null_amount_count'].values[0] > 0:
        print("Null values found in 'amount' column.")

#### Closing Summary
and Common Mistakes

By following these steps, we ensure that the nightly load into BigQuery is validated for both data integrity and completeness. The key mistakes to avoid include missing initial source validation, not confirming successful loading, and ignoring post-loading checks.

Common Follow-Ups
What tools do you use for monitoring ETL processes?
How would you handle errors during the load process?
Can you explain how BigQuery's partitioning and clustering features play a role in this scenario?

**Comparison:**

| Column    | Description                                      |
|-----------|--------------------------------------------------|
| user_id   | Unique identifier for each transaction           |
| transaction_date | Date of the transaction                          |
| amount    | Transaction amount                               |

**Code:**

```python
import pandas as pd

def validate_source_data(source_df):
    required_columns = ['user_id', 'transaction_date', 'amount']
    missing_cols = [col for col in required_columns if col not in source_df.columns]

    if missing_cols:
        raise ValueError(f"Missing columns: {missing_cols}")

    try:
        pd.to_datetime(source_df['transaction_date'], errors='raise')
    except ValueError as e:
        print(f"Date validation failed: {e}")
        return False

    return True

def load_data_to_bigquery(source_df):
    client = bigquery.Client()

    dataset_id = "my_dataset"
    table_id = f"{dataset_id}.transactions"

    job_config = bigquery.LoadJobConfig(
        schema=[
            bigquery.SchemaField("user_id", "STRING"),
            bigquery.SchemaField("transaction_date", "DATE"),
            bigquery.SchemaField("amount", "NUMERIC")
        ],
        skip_leading_rows=1,
        write_disposition="WRITE_TRUNCATE"
    )

    job = client.load_table_from_dataframe(source_df, table_id, job_config=job_config)
    job.result()  # Waits for the load job to complete

    return True

def validate_loaded_data():
    client = bigquery.Client()

    query = """
    SELECT COUNT(*) AS total_rows, COUNT(user_id) AS user_count,
           COUNTIF(amount IS NULL) AS null_amount_count
    FROM `my_dataset.transactions`
    """

    query_job = client.query(query)
    results = query_job.result().to_dataframe()

    expected_row_count = 100000  # Example expected value

    if results['total_rows'].values[0] != expected_row_count:
        print(f"Row count mismatch: Expected {expected_row_count}, got {results['total_rows'].values[0]}")

    if results['null_amount_count'].values[0] > 0:
        print("Null values found in 'amount' column.")
```

**Example input / output:**

```text
Input:
- Source DataFrame with user_id, transaction_date, and amount columns

Output:
- Total rows count in BigQuery table matches the expected value (e.g., 100,000)
- No null values found in critical columns like 'amount'
```

**Data:**

```json
{
  "user_id": ["123", "456"],
  "transaction_date": ["2023-09-15", "2023-09-16"],
  "amount": [100.0, 200.0]
}
```

**Flow:**

1. Pass
2. Yes
3. No
4. Start
5. Validate Source Data
6. Load to BigQuery
7. Confirm Load Success
8. Post-Load Checks
9. End Validation Process
10. Handle Errors and Retry

