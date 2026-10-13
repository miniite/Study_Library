---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a landing zone, a raw zone, and a curated zone in a data lake pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Checkpointing and Recovery](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Checkpointing_and_Recovery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a data lake architecture, zones like landing, raw, and curated serve distinct purposes in managing the lifecycle of data as it moves from ingestion to consumption. A quick overview:

Landing Zone: Raw, unprocessed data is first stored here.
Raw Zone: Data remains in its original form but undergoes minimal processing (e.g., renaming files, decompression).
Curated Zone: Processed and cleansed data that's ready for analytics or reporting.

To dive into these concepts from the ground up:

Understanding Zones:

The landing zone is where raw data lands initially.
The raw zone processes data to a basic form without altering its structure significantly.
The curated zone handles complex transformations and cleanses the data, making it suitable for analytics.

#### Triaging the Question

The interviewer likely wants you to explain the roles of each zone and how they fit into the overall data pipeline architecture.

#### Assumptions
We are discussing a typical data lake setup with three distinct zones as described above.

#### First-Principles Solution

Landing Zone: This is where raw, unprocessed data first lands in your system. It’s crucial because it ensures that all data is captured without any preprocessing.
PYTHON
# Example: Saving raw logs to S3 landing zone
s3_client.upload_file(log_file_path, 'landing-zone-bucket', log_file_name)
Raw Zone: Here, the data is slightly processed—possibly decompressed or renamed for better handling. This step is necessary because raw data might be in compressed formats or unstructured.
PYTHON
# Example: Decompressing logs and renaming files
def decompress_and_rename_files(bucket, prefix):
    s3 = boto3.client('s3')
    objects = s3.list_objects_v2(Bucket=bucket, Prefix=prefix)['Contents']
    for obj in objects:
        key = obj['Key']
        # Decompress the file if needed
        if '.gz' in key:
            decompressed_key = key.replace('.gz', '')
            s3.download_file(bucket, key, '/tmp/tmp.gz')
            with open('/tmp/tmp.gz', 'rb') as f_in, gzip.open('/tmp/decompressed.gz', 'wb') as f_out:
                shutil.copyfileobj(f_in, f_out)
            s3.upload_file('/tmp/decompressed.gz', bucket, decompressed_key)
        # Rename the file if needed
        new_key = key.replace('_old', '_new')
        s3.rename_object(CopySource=f'{bucket}/{key}', Bucket=bucket, Key=new_key)
Curated Zone: In this zone, data is fully processed and cleansed. It’s where you apply business rules, transformations, and validations to ensure the data is fit for consumption.
PYTHON
# Example: Data transformation in curated zone
def transform_data(df):
    df = df.dropna()  # Remove missing values
    df['date'] = pd.to_datetime(df['date'])  # Ensure date column is a datetime object
    df = df[df['amount'] > 0]  # Filter out negative amounts
    return df

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The key takeaway is that these zones help in managing different stages of data processing, from raw to curated.

#### Common mistakes
might include skipping the landing zone, not properly handling raw data, or failing to validate and cleanse data sufficiently in the curated zone.

**Comparison:**

| Zone       | Description                                         | Processing Level      |
|------------|-----------------------------------------------------|-----------------------|
| Landing    | Raw, unprocessed data first lands here              | Minimal                |
| Raw        | Data is slightly processed (e.g., decompression)   | Basic                  |
| Curated    | Fully processed and cleansed for analytics         | Complex                 |

**Code:**

```python
# Example code snippets
```

**Example input / output:**

```text
# Example input to output flow
Input:
- Raw data files in 'raw-data-bucket/raw/'

Processing Steps:
1. Upload raw data to S3 (Landing Zone).
2. Decompress and rename files (Raw Zone).
3. Apply transformations and validations (Curated Zone).

Output:
- Processed, cleansed data stored in 'analyzed-data-bucket/clean/'
```

**Data:**

```json
{
  "landing_zone": {
    "bucket_name": "raw-data-bucket",
    "prefix": "raw/"
  },
  "raw_zone": {
    "bucket_name": "processed-data-bucket",
    "prefix": "intermediate/"
  },
  "curated_zone": {
    "bucket_name": "analyzed-data-bucket",
    "prefix": "clean/"
  }
}
```

**Flow:**

1. Landing Zone
2. Raw Zone
3. Curated Zone

