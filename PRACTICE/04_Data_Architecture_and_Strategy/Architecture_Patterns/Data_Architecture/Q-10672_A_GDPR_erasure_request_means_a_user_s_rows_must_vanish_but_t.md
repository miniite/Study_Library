---
type: "practice"
difficulty: "Deep"
---

# A GDPR erasure request means a user's rows must vanish, but they're baked into your training set. How does this affect your data architecture and model lifecycle?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Immutable Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/GDPR_Deletion_in_Immutable_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary:

In response to a GDPR erasure request, ensuring that a user's data is fully removed from both the raw data lake and the production model can be challenging. This requires careful planning in your data architecture to support efficient and safe deletion of data while maintaining the integrity of ongoing machine learning (ML) training and inference processes.

#### Triaging the Question

The question is probing how to handle GDPR compliance, specifically erasure requests, within an ML environment that uses a combination of data lakes and data warehouses. The key aspects include:

Data Storage: Understanding where user data resides in your architecture (raw data lake vs. processed data warehouse).
Model Training: Ensuring that removing user data from the training set doesn't compromise model performance.
Inference Impact: Managing how deleted data affects ongoing or new inference requests.

#### Assumptions
and First Principles:
Data Lakes: Stores raw, unprocessed data.
Data Warehouses: Processed and cleaned data used for analysis and reporting.
Model Training: Uses both raw and processed data depending on the training pipeline.
Inference: Directly uses the model that is trained and may incorporate features from either the lake or warehouse.
Detailed Solution:
1. Data Architecture Design

To handle GDPR erasure requests effectively, your architecture should include mechanisms to identify and remove user-specific data efficiently:

Data Lake (Raw Data):

Metadata Management: Implement metadata tags to identify which rows correspond to specific users.
Delete Mechanism: Develop a robust delete pipeline that can quickly remove these tagged records.
Yes
No
User Data in Raw Data Lake
(Tag User Rows)
Metadata Tags for User Data
Is Deletion Requested?
Delete Tagged Rows from RawData Lake
Continue Monitoring

Data Warehouse (Processed Data):

Anonymization: Ensure that processed data does not contain direct identifiers.
Join Mechanism: Use a join mechanism to remove user-specific data when merging with other datasets.
Yes
No
User Data in Raw Data Lake
(Tag User Rows)
Metadata Tags for User Data
Is Deletion Requested?
Remove Tagged Rows fromProcessed Data Warehouse
Continue Monitoring
2. Model Training and Retraining

To ensure that the model remains effective after a data erasure request:

Training Set Update: Design your training pipeline to dynamically update the training set with new or updated data.
Retraining Strategy: Implement a retraining mechanism to refresh the model with the latest, clean data.
Yes
User Data in Raw Data Lake
(Tag User Rows)
Metadata Tags for User Data
Is Deletion Requested?
Update Training Set with NewData
Retrain Model with UpdatedTraining Set
3. Inference Management

For inference, ensure that the model uses only the latest, clean data:

Inference Pipeline: Implement a pipeline that fetches the most recent version of the model and the associated datasets.
Cache Management: Use caching to reduce latency but manage it carefully to avoid stale data.
Yes
User Data in Raw Data Lake
(Tag User Rows)
Metadata Tags for User Data
Is Deletion Requested?
Update Model and InferenceDatasets
Infer Using Updated Modeland Clean Data
4. Monitoring and Compliance
Audit Logs: Maintain logs to track data deletions and model retraining.
Compliance Checks: Regularly check that your architecture adheres to GDPR guidelines.
Yes
User Data in Raw Data Lake
(Tag User Rows)
Metadata Tags for User Data
Is Deletion Requested?
Update Training Set with NewData
Retrain Model with UpdatedTraining Set
Audit Logs and ComplianceChecks

#### Closing Summary

Handling GDPR erasure requests in an ML environment requires a well-designed data architecture that can efficiently remove user-specific data from both the raw data lake and processed data warehouse. This involves implementing metadata tagging, robust deletion pipelines, dynamic retraining strategies, and ensuring compliance through regular audits.

#### Common Mistakes/Follow-Ups
Data Inconsistency: Failing to ensure consistent data removal across all relevant datasets.
Model Degradation: Not properly updating the training set or failing to retrain the model can lead to degraded performance.
Latency Issues: Inadequate caching strategies can introduce significant delays in inference.

**Comparison:**

| User ID | Tags                |
|---------|---------------------|
| 12345   | personal_info       |
|         | sensitive_data      |

**Code:**

```python
# Example of a simple delete pipeline in Python
def remove_user_data(user_id):
    raw_data_lake.delete_records_by_tag(user_id)
    processed_data_warehouse.remove_joined_rows(user_id)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Erasure request for user ID 12345 with tags ["personal_info", "sensitive_data"] | Removed tagged rows from raw data lake and processed data warehouse |

**Data:**

```json
{
  "user_id": "12345",
  "tags": ["personal_info", "sensitive_data"]
}
```

**Flow:**

1. Yes
2. No
3. User Data in Raw Data Lake
4. (Tag User Rows)
5. Metadata Tags for User Data
6. Is Deletion Requested?
7. Delete Tagged Rows from RawData Lake
8. Continue Monitoring

