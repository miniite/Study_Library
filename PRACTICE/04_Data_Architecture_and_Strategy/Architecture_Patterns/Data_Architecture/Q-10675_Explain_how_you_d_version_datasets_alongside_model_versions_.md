---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd version datasets alongside model versions so you can always trace which data produced which model.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Reproducible Training Data Versions](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Reproducible_Training_Data_Versions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure traceability between model versions and their corresponding input datasets, we need to establish a robust data architecture that includes versioning mechanisms for both models and datasets. This involves designing a system where each dataset has a unique identifier and version number, and these are linked with the model versions used during training or inference. The key challenge is to maintain consistency in the metadata linking these elements across different stages of the ML pipeline.

In this context, let's consider an example architecture that integrates data lakes, warehouses, and a mesh network for efficient data processing and storage.

#### Triaging the Question

The question essentially asks how to implement versioning in datasets to ensure they are traceable with model versions. The interviewer is probing your understanding of data management practices, especially in complex ML pipelines where multiple models might use different data versions at various points in time. We'll assume a scenario where we have both offline storage (data lake) and online query-optimized storage (data warehouse), along with real-time processing capabilities.

#### First Principles Solution
Data Lake: Store raw, unprocessed data.
Data Warehouse: Processed and aggregated data for querying and reporting.
Versioning Mechanism: Ensure datasets and models are versioned properly to maintain traceability.
Metadata Management: Maintain a metadata layer that links datasets and model versions.
Detailed Solution

Versioned Datasets in Data Lake

Each dataset is stored with a unique identifier (e.g., UUID) and version number.
JSON
{
  "uuid": "12345678-90ab-cdef-fedc-ba9876543210",
  "version": 1,
  "data": [...]
}
Use a version control system (e.g., Git) to manage changes in the data schema and structure.

Versioned Models

Store model artifacts with their own unique identifiers and versions.
JSON
{
  "model_id": "98765432-10ab-cdef-fedc-ba9876543210",
  "version": 1,
  "architecture": {...},
  "weights": [...]
}

Metadata Management

Maintain a metadata database (e.g., using PostgreSQL, MySQL) that links datasets and models.
SQL
CREATE TABLE dataset_model_links (
    id SERIAL PRIMARY KEY,
    dataset_uuid VARCHAR(36),
    model_id VARCHAR(36),
    training_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Data Ingestion Pipeline

When a new version of data is ingested, it gets stored with its version number.
The pipeline also records the model ID and timestamp for when this dataset was used.

Model Training/Inference Pipeline

During training or inference, the system retrieves the correct dataset based on the specified version.
Record the metadata in the dataset_model_links table to establish the link between the data and model versions.

Retrieval Mechanism

When a model is queried for its input data, use the metadata stored in the dataset_model_links table to retrieve the correct dataset version.
SQL
SELECT * FROM dataset_model_links WHERE model_id = '98765432-10ab-cdef-fedc-ba9876543210' ORDER BY training_timestamp DESC LIMIT 1;

#### Closing Summary
and Common Mistakes/Follow-Ups

The key to successfully versioning datasets alongside model versions is maintaining a clear metadata layer that tracks all dependencies. This ensures that any given model can be traced back to the exact input data used during its training or inference.

#### Common Mistakes

Failing to implement proper version control for both datasets and models.
Not using a robust metadata management system, leading to inconsistencies in tracking.
Ignoring the importance of timestamping to ensure chronological traceability.

#### Follow-Ups

How do you handle different environments (production, staging)?
What strategies do you use for managing large-scale data?
How do you ensure data lineage and auditability?

**Comparison:**

```
Field	Description
dataset_uuid	Unique identifier for the dataset
model_id	Unique identifier for the model
training_timestamp	Timestamp when the link was established
```

**Code:**

```

{ "uuid": "12345678-90ab-cdef-fedc-ba9876543210", "version": 1, "data": [...] }

CODE
Copy
```json
{
  "model_id": "98765432-10ab-cdef-fedc-ba9876543210",
  "version": 1,
  "architecture": {...},
  "weights": [...]
}
```

**Example input / output:**

```text
-- Example of retrieving the latest data version used for a model
SELECT * FROM dataset_model_links WHERE model_id = '98765432-10ab-cdef-fedc-ba9876543210' ORDER BY training_timestamp DESC LIMIT 1;
```

**Data:**

```sql
CREATE TABLE dataset_model_links (
    id SERIAL PRIMARY KEY,
    dataset_uuid VARCHAR(36),
    model_id VARCHAR(36),
    training_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Is Data Versioned?
5. Store in Data Lake
6. Update Metadata
7. Record Model ID andTimestamp
8. Model Training/Inference
9. Retrieve Metadata
10. Fetch Correct Dataset Version

