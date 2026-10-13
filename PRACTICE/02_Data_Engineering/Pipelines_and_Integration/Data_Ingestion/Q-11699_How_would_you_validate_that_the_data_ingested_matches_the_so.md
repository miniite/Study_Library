---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate that the data ingested matches the source (row counts, checksums)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Ingestion Credentials](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Ingestion_Credentials.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the data ingested using Airbyte or Fivetran matches the original source, we need to perform several validation steps. These include verifying row counts and performing checksum comparisons. Let's break down the process step by step.

Firstly, we need to understand what these validations entail. Row count verification involves comparing the number of rows in the source with those in the target system post-ingestion. Checksums provide a more granular comparison by hashing the data before and after ingestion, ensuring that even small changes are detected.

Now, let's discuss how this can be implemented using Airbyte:

Setup Airbyte Configuration: Configure the pipeline to connect your source (e.g., MySQL) with your target (e.g., PostgreSQL). This involves creating a new connection in the Airbyte UI and defining the appropriate sources and targets.

Row Count Validation:

Step 1: Before ingestion, record the row count of the source table using SQL queries or API calls.
SQL
SELECT COUNT(*) AS source_row_count FROM source_table;
Step 2: After Airbyte completes the data transfer and loads the data into the target database, execute a similar query to get the target row count.
SQL
SELECT COUNT(*) AS target_row_count FROM target_table;
Step 3: Compare these counts. If they match, it suggests that no rows were lost or duplicated during the transfer.

Checksum Validation:

Step 1: Generate a checksum for the source data before ingestion.
PYTHON
import hashlib

# Assuming we are working with a single column 'data' in a pandas DataFrame
def generate_checksum(data):
    return hashlib.sha256(data.encode()).hexdigest()

original_checksum = generate_checksum(source_table['data'])
Step 2: After the data is ingested into the target system, pull the same data and generate a new checksum.
PYTHON
# Assuming we have a DataFrame 'target_data' from the target table
target_checksum = generate_checksum(target_data['data'])
Step 3: Compare the two checksums. If they match, it indicates that no data was altered during the transfer.

