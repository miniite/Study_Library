---
type: "practice"
difficulty: "Basic"
---

# How would you load data into Google BigQuery from Python, and what are the basic steps to run a load job or streaming insert?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming Query Results](<../../../../TOOLS/Python/Python_Language/Streaming_Query_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To load data into Google BigQuery from Python, we need to follow a series of steps that involve setting up the necessary environment, installing required libraries, and writing Python code to execute BigQuery jobs. The basic concept revolves around using the google-cloud-bigquery library, which is officially supported by Google for interacting with BigQuery through Python.

#### Triage

The question asks about loading data into Google BigQuery from Python, specifically mentioning load jobs or streaming inserts. This means we need to cover both batch and real-time data ingestion methods. The core idea here is to use the google-cloud-bigquery library effectively, handle authentication properly, and understand the differences between load jobs and streaming inserts.

Steps to Run a Load Job
Install Required Libraries: Ensure you have the necessary Python libraries installed.
Set Up Authentication: Use service account credentials for secure access.
Initialize BigQuery Client: Create a client object using your credentials.
Prepare Data: Format your data according to BigQuery requirements.
Execute Load Job: Use the appropriate methods to load the data into BigQuery.
Steps to Run Streaming Inserts
Install Required Libraries: Ensure you have the necessary Python libraries installed.
Set Up Authentication: Use service account credentials for secure access.
Initialize BigQuery Client: Create a client object using your credentials.
Prepare Data: Format your data according to BigQuery requirements.
Execute Streaming Insert: Use the appropriate methods to insert data in real-time.
Detailed Solution

Install Required Libraries

SH
pip install google-cloud-bigquery pandas

Set Up Authentication

Download a JSON key file from the Google Cloud Console for your service account.
Set environment variables or provide credentials directly:
PYTHON
import os
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/path/to/service-account-file.json"

Initialize BigQuery Client

PYTHON
from google.cloud import bigquery

client = bigquery.Client()

Prepare Data

For load jobs, you can use a pandas DataFrame or CSV file.

#### Example
using a pandas DataFrame:
PYTHON
import pandas as pd

df = pd.read_csv("data.csv")

Execute Load Job

PYTHON
dataset_id = 'your_dataset_id'
table_id = 'your_table_id'

job_config = bigquery.LoadJobConfig(
    autodetect=True,
    write_disposition='WRITE_TRUNCATE',
    source_format=bigquery.SourceFormat.CSV
)

load_job = client.load_table_from_dataframe(df, f'{dataset_id}.{table_id}', job_config=job_config)

load_job.result()  # Wait for the job to complete

print(f'Loaded {load_job.output_rows} rows into {dataset_id}:{table_id}')
Streaming Inserts
Initialize BigQuery Client (same as above)
Prepare Data

#### Example
data:
PYTHON
insert_data = [
    {"name": "John", "age": 30},
    {"name": "Jane", "age": 25}
]
Execute Streaming Insert
PYTHON
table_ref = client.dataset(dataset_id).table(table_id)
errors = client.insert_rows_json(table_ref, insert_data)  # Make an API request

if not errors:
    print(f'Successfully inserted rows into {dataset_id}:{table_id}')
else:
    print('Encountered errors while inserting rows:', errors)

#### Closing Summary

In summary, loading data into BigQuery from Python involves setting up the environment, authenticating with Google Cloud, and using the google-cloud-bigquery library to execute either load jobs or streaming inserts. Load jobs are ideal for large datasets, whereas streaming inserts allow real-time data ingestion.

#### Common Mistakes/Follow-Ups
Mistake: Not handling authentication properly can lead to permission issues.

#### Follow-Up
What happens if the data format is not compatible with BigQuery? You may need to preprocess the data or adjust the schema.

**Code:**

```
Load job example using pandas DataFrame

import pandas as pd from google.cloud import bigquery

os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/path/to/service-account-file.json"

client = bigquery.Client()

df = pd.read_csv("data.csv")

job_config = bigquery.LoadJobConfig( autodetect=True, write_disposition='WRITE_TRUNCATE', source_format=bigquery.SourceFormat.CSV )

load_job = client.load_table_from_dataframe(df, 'your_dataset_id.your_table_id', job_config=job_config) load_job.result()

CODE
Copy

```python
# Streaming insert example
from google.cloud import bigquery

os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/path/to/service-account-file.json"

client = bigquery.Client()

table_ref = client.dataset('your_dataset_id').table('your_table_id')
errors = client.insert_rows_json(table_ref, [{"name": "John", "age": 30}, {"name": "Jane", "age": 25}])
if not errors:
    print("Successfully inserted rows")
else:
    print("Encountered errors while inserting rows:", errors)
```

**Example input / output:**

```

Load Job Example:

PYTHON
Copy
import pandas as pd
from google.cloud import bigquery

os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/path/to/service-account-file.json"

client = bigquery.Client()

df = pd.read_csv("data.csv")

job_config = bigquery.LoadJobConfig(
    autodetect=True,
    write_disposition='WRITE_TRUNCATE',
    source_format=bigquery.SourceFormat.CSV
)

load_job = client.load_table_from_dataframe(df, 'your_dataset_id.your_table_id', job_config=job_config)
load_job.result()

print(f'Loaded {load_job.output_rows} rows into your_dataset_id:your_table_id')

Streaming Insert Example:

PYTHON
Copy
from google.cloud import bigquery

os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "/path/to/service-account-file.json"

client = bigquery.Client()

table_ref = client.dataset('your_dataset_id').table('your_table_id')
errors = client.insert_rows_json(table_ref, [{"name": "John", "age": 30}, {"name": "Jane", "age": 25}])

if not errors:
    print("Successfully inserted rows")
else:
    print("Encountered errors while inserting rows:", errors)
```

**Data:**

```json
{
  "data.csv": [
    ["name", "age"],
    ["John", 30],
    ["Jane", 25]
  ]
}
```

