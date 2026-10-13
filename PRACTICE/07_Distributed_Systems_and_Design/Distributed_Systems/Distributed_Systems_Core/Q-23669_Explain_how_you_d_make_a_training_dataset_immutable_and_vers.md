---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd make a training dataset immutable and versioned so an experiment is exactly reproducible six months later. What does data lineage give you when a model in production is found to have trained on corrupted data?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event Schema Evolution](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a training dataset used for experiments remains immutable and versioned, enabling exact reproducibility six months later, we need to implement a comprehensive system that captures every aspect of the dataset's lifecycle, from creation to its use in training models. This involves making data read-only once it has been finalized, tracking all changes through versions, and maintaining a clear lineage of how each dataset was derived.

At a high level, this solution entails:

Data Versioning: Assign unique identifiers (versions) to each state of the dataset.
Immutable Storage: Ensure that once data is committed to storage, it cannot be altered.
Lineage Tracking: Maintain detailed records of transformations applied to datasets and dependencies among them.

Now, let's delve into the first-principles solution for making a training dataset immutable and versioned:

#### Triaging the Question

The core of this question involves ensuring that experiments are reproducible over long periods by managing data immutability and lineage. The interviewer is probing your understanding of distributed systems, version control, and robust data management practices.

#### Assumptions
We have a well-defined data pipeline.
Data transformations can be logged and tracked.
There's an existing versioning system for datasets.
Step-by-Step Solution
Data Versioning:
Each dataset should have a unique identifier that includes the timestamp, version number, and any other relevant metadata (e.g., source data, transformations applied).
PLAINTEXT
Dataset_v2023_05_01_1200
Immutable Storage:
Use immutable storage mechanisms like object storage services that support versioning.
Ensure that once a dataset is written to this service, it cannot be altered.
PYTHON
# Example using S3 with versioning enabled
import boto3

s3 = boto3.client('s3', region_name='us-west-2')
response = s3.put_bucket_versioning(Bucket='my-bucket', VersioningConfiguration={'Status': 'Enabled'})
Lineage Tracking:
Maintain a metadata store that logs the lineage of each dataset.
Record all transformations applied to datasets, including source data, transformations used, and any intermediate states.
PLAINTEXT
Metadata Store Schema:
| Dataset ID | Source Data Version | Transformations Applied | Time Stamp |
|------------|--------------------|------------------------|------------|
| 1001       | v2023_04_30        | CleanseData, FeatureExtract | 2023-05-01T12:00Z |
Audit Logs:
Implement logging for all read and write operations on the dataset.
Ensure logs are stored in a secure, immutable location.
PYTHON
# Example using AWS CloudTrail to log S3 operations
cloudtrail = boto3.client('cloudtrail', region_name='us-west-2')
response = cloudtrail.start_logging()
Testing and Validation:
Regularly test the reproducibility of experiments by re-running models on historical datasets.
Ensure that all dependencies are correctly versioned to avoid any unintended changes.

#### Closing Summary

Implementing a robust system for making training datasets immutable and versioned requires a combination of technical practices such as using immutable storage, maintaining detailed lineage records, and ensuring thorough testing. This approach not only guarantees reproducibility but also provides valuable insights into the evolution of data over time.

#### Common Mistakes
and Follow-Ups
Ignoring Data Transformations: Ensuring that all transformations applied to datasets are logged.
Inadequate Logging: Failing to maintain logs for all read/write operations on datasets, leading to difficulty in tracing changes.
Siloed Systems: Not integrating lineage tracking with other parts of the data pipeline, which can lead to inconsistencies.

To further explore this topic, you could discuss how distributed systems handle consensus and fault tolerance in maintaining dataset versions. Additionally, exploring advanced techniques like differential storage or incremental backups might be beneficial for optimizing storage and bandwidth usage.

**Comparison:**

| Dataset ID | Source Data Version | Transformations Applied | Time Stamp |
|------------|--------------------|------------------------|------------|
| 1001       | v2023_04_30        | CleanseData, FeatureExtract | 2023-05-01T12:00Z |

**Code:**

```python
# Example using S3 with versioning enabled
import boto3

s3 = boto3.client('s3', region_name='us-west-2')
response = s3.put_bucket_versioning(Bucket='my-bucket', VersioningConfiguration={'Status': 'Enabled'})
```

**Example input / output:**

```text
Input:
- Original dataset version: v2023_04_30

Output:
- Finalized and immutable version of the dataset: Dataset_v2023_05_01_1200
- Metadata record created:
  - Source Data Version: v2023_04_30
  - Transformations Applied: CleanseData, FeatureExtract
  - Time Stamp: 2023-05-01T12:00Z
```

**Data:**

```plaintext
Metadata Store Schema:
| Dataset ID | Source Data Version | Transformations Applied | Time Stamp |
|------------|--------------------|------------------------|------------|
| 1001       | v2023_04_30        | CleanseData, FeatureExtract | 2023-05-01T12:00Z |
```

**Flow:**

1. Yes
2. No
3. Dataset Creation
4. Is Dataset Finalized?
5. Write to Immutable Storage
6. Transformations Applied
7. Create Metadata Record
8. Log All Operations in SecureLocation

