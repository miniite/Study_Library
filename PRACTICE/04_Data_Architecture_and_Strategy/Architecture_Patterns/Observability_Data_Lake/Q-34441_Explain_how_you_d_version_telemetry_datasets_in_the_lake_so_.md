---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd version telemetry datasets in the lake so an anomaly model can be reproducibly retrained on the exact data it saw.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Curated Observability Datasets](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Curated_Observability_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure reproducible retraining of anomaly models, we need to maintain a versioned history of telemetry datasets in our Observability Data Lake. This involves storing each dataset with a unique identifier and timestamp, ensuring that historical data can be retrieved exactly as it was when the model was trained. The key idea here is to create an immutable log of all inputs used by the anomaly detection process.

The question probes how we manage data lineage, version control, and reproducibility in the context of machine learning models. We need to make assumptions about the structure and format of the telemetry data and the storage mechanisms available. For this explanation, I will assume a JSON-based schema for simplicity, but the principles can be adapted to various formats.

Data Versioning: Each telemetry dataset should have a unique version ID that includes metadata such as timestamp, environment (e.g., production vs. staging), and any other relevant identifiers.
Storage Mechanism: We will use a structured storage system like Amazon S3 or Azure Blob Storage to ensure durability and scalability.
Metadata Management: A separate metadata store, possibly a database like AWS DynamoDB, will keep track of the relationships between datasets and their versions.

Let's walk through the steps from first principles:

Data Collection & Transformation:
Telemetry data is collected in real-time from various sources (e.g., servers, network devices).
This raw data is processed to extract relevant metrics.
Versioning:
Each batch of transformed telemetry data gets a version ID. For example: v12345_2023-09-15T16:00:00Z.
The version ID includes the timestamp and a unique identifier for the specific dataset.
Storage:
Processed data is stored in a version-controlled manner. Each version of the data has its own subdirectory (e.g., v12345).
Metadata about each version, including details like environment and source, is stored separately.
Retrieval & Reproduction:
When retraining an anomaly model, we retrieve the exact dataset version used in training by querying the metadata store.
The anomaly detection model is then retrained using this historical data.

By following these steps, we ensure that every retraining session uses the same input data as the original, allowing for reproducible results. This approach also enables us to maintain a clear audit trail and facilitates debugging if issues arise with the model.

#### Common mistakes
include:

Not properly versioning datasets, leading to confusion when multiple versions are used.
Failing to document metadata accurately, making it hard to track which data was used in training.
Using inconsistent or poorly structured storage mechanisms that complicate retrieval of historical data.

**Comparison:**

| Version ID        | Environment | Source          |
|------------------|-------------|----------------|
| v12345_2023-09-15T16:00:00Z | production  | server_logs    |

**Code:**

```python
import os

def store_telemetry_data(version_id, telemetry_data):
    """Store telemetry data with a unique version ID."""
    base_dir = "/path/to/data/lake"
    version_dir = os.path.join(base_dir, version_id)

    # Ensure the directory exists
    if not os.path.exists(version_dir):
        os.makedirs(version_dir)

    # Store the data in the versioned directory
    with open(os.path.join(version_dir, "data.json"), 'w') as f:
        json.dump(telemetry_data, f)

def get_version_metadata(version_id):
    """Retrieve metadata for a specific version."""
    metadata_db = boto3.client('dynamodb')
    response = metadata_db.get_item(
        TableName='TelemetryMetadata',
        Key={'VersionId': {'S': version_id}}
    )

    return response['Item']
```

**Example input / output:**

```

Input:

PYTHON
Copy
telemetry_data = {
    "timestamp": "2023-09-15T16:00:00Z",
    "cpu_usage": [80, 75, 82],
    "memory_usage": [45, 50, 48]
}

store_telemetry_data("v12345_2023-09-15T16:00:00Z", telemetry_data)

Output:

PLAINTEXT
Copy
Data stored in directory /path/to/data/lake/v12345_2023-09-15T16:00:00Z/data.json
Metadata stored in DynamoDB with version_id: v12345_2023-09-15T16:00:00Z, environment: production, source: server_logs
```

**Data:**

```json
{
  "version_id": "v12345_2023-09-15T16:00:00Z",
  "environment": "production",
  "source": "server_logs"
}
```

**Flow:**

1. Yes
2. No
3. Telemetry Data Collection
4. Is Version ID Unique?
5. Store Telemetry Data &Metadata
6. Generate Unique Version ID
7. Return Storage Directory Path

