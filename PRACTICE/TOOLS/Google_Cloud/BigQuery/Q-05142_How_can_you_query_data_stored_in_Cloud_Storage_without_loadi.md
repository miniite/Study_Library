---
type: "practice"
difficulty: "Intermediate"
---

# How can you query data stored in Cloud Storage without loading it into BigQuery?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ BigQuery Loading and Table Design](<../../../../TOOLS/Google_Cloud/BigQuery/BigQuery_Loading_and_Table_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To query data stored in Google Cloud Storage (GCS) without loading it into BigQuery, we need to leverage the powerful capabilities of BigQuery's external data source feature. This approach allows us to write SQL queries directly against files in GCS, such as CSV or Parquet files, which is incredibly efficient and convenient for large datasets.

BigQuery supports reading from various file formats like JSON, Avro, ORC, and Parquet, making it flexible enough to handle a wide variety of data storage needs. However, we must ensure that the data in GCS is correctly formatted according to BigQuery's requirements. Let’s break down how this can be achieved step by step.

**Comparison:**

| id | name  | age | salary |
|----|-------|-----|--------|
| 1  | Alice | 30  | 50000  |
| 2  | Bob   | 45  | 60000  |

**Code:**

```python
# Example Python code snippet to query GCS files using BigQuery
from google.cloud import bigquery

def query_gcs_data(bucket_name, file_path):
    client = bigquery.Client()

    # Construct a BigQuery job configuration.
    query_job_config = bigquery.QueryJobConfig(
        query_parameters=[
            bigquery.ScalarQueryParameter("bucket", "STRING", bucket_name),
            bigquery.ScalarQueryParameter("file_path", "STRING", file_path),
        ]
    )

    sql_query = f"""
    SELECT *
    FROM `{bucket_name}:{file_path}`
    """

    # Start the query, passing in the extra configuration.
    query_job = client.query(sql_query, job_config=query_job_config)

    # Wait for the query to finish and fetch results
    results = query_job.result()
    return list(results)
```

**Example input / output:**

```

Input:

PYTHON
Copy
query_gcs_data("my-gcs-bucket", "data/2023-10-01.csv")

Output:

CODE
Copy
[Row(id='1', name='Alice', age=30, salary=50000),
 Row(id='2', name='Bob', age=45, salary=60000)]
```

**Data:**

```json
{
  "bucket_name": "my-gcs-bucket",
  "file_path": "data/2023-10-01.csv"
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is data in GCS?
5. Use BigQuery External Table
6. Write SQL query to GCS filepath
7. Execute query and fetch results
8. Load data into BigQuery
9. Store results if needed

