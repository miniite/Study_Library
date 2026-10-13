---
type: "practice"
difficulty: "Deep"
---

# How do you architect a data platform for a genomics/research org where individual datasets are terabyte-scale files (BAM/VCF) that don't fit a columnar table model? Where does object-storage-plus-metadata-index beat a warehouse, and how do you still provide catalog, lineage, and governance over opaque files?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Vector Data in the Lakehouse](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Vector_Data_in_the_Lakehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting a data platform for a genomics/research organization with terabyte-scale datasets like BAM and VCF files, it's crucial to balance scalability, manageability, and cost-effectiveness. The key challenge is handling large, irregularly structured files that don't fit into traditional relational databases. In this scenario, object storage combined with metadata indexing can provide a flexible and scalable solution while maintaining catalog, lineage, and governance.

TL;DR

Object storage plus metadata index beats a traditional warehouse when dealing with terabyte-scale datasets like BAM/VCF because it offers better scalability, flexibility, and cost-efficiency. However, to ensure effective management and governance, you need robust tools for cataloging, lineage tracking, and data validation.

#### Triaging the Question

The question is probing how we can design a robust data platform that handles large, unstructured datasets efficiently while maintaining critical metadata and governance. We need to consider:

Scalability: The ability to handle vast amounts of data without performance degradation.
Flexibility: Managing various file formats and structures.
Cost-effectiveness: Balancing storage costs with the need for advanced analytics.
Governance: Ensuring compliance, trackability, and security of stored data.

#### First-Principles Solution

Object Storage as the Foundation

Data Repository: Use object storage services (e.g., AWS S3, Azure Blob Storage) to store raw BAM and VCF files. Object storage is designed for high scalability and durability.

Metadata Indexing for Cataloging

Catalog Service: Implement a metadata catalog service that maintains detailed information about each file, including lineage, provenance, and schema details.
Indexing Mechanism: Use an indexing layer to provide efficient query capabilities over the stored files without needing to load them entirely into memory.

Data Processing Pipelines

ETL Processes: Develop ETL (Extract, Transform, Load) pipelines using tools like Apache Airflow or AWS Glue to automate data ingestion and preprocessing.
Compute Resources: Use serverless computing services (e.g., AWS Lambda) for lightweight operations that can be triggered on-demand.

Governance and Security

Access Control: Implement strict access controls based on roles and permissions using Identity and Access Management (IAM).
Audit Logs: Enable logging to track all data access and modifications.
Compliance Tools: Use tools like AWS Config or Azure Policy to ensure compliance with regulatory standards.
Detailed Steps

Data Ingestion

Storage Service: Store BAM and VCF files in object storage, ensuring they are versioned for durability.
Metadata Collection: During ingestion, collect detailed metadata about each file (e.g., creation time, user ID, data provenance).

Indexing Layer

Metadata Database: Use a distributed database (e.g., Cassandra) to store and query the metadata efficiently.
Catalog Service: Develop a service that provides RESTful APIs for querying and managing the metadata.

Data Processing Pipelines

ETL Jobs: Define ETL jobs using tools like AWS Glue or Apache Airflow to process incoming data into more structured formats suitable for analytics.
Compute Resources: Use serverless functions (e.g., AWS Lambda) for tasks that don’t require persistent resources.

Governance and Security

IAM Policies: Define IAM policies to control who can read/write data in the storage layer.
Audit Trails: Enable audit trails on all storage and processing services to track access and modifications.
Compliance Checks: Regularly run compliance checks using tools like AWS Config or Azure Policy.

#### Closing Summary

In summary, object storage combined with metadata indexing provides a scalable and flexible solution for managing terabyte-scale datasets in genomics research. By leveraging robust cataloging, data processing pipelines, and governance mechanisms, we can ensure efficient data management while maintaining stringent security and compliance requirements.

#### Common Mistakes/Follow-Ups
Ignoring Metadata Management: Not adequately tracking lineage and provenance can lead to mismanagement of data.
Inadequate Security Measures: Failing to implement strict access controls and audit trails can compromise data integrity.
Overlooking Data Governance: Ignoring compliance checks and policies can result in legal and operational risks.

**Comparison:**

| File ID | Filename        | Size   | Format  | Timestamp           | Provenance Source       | Provenance User ID    |
|---------|-----------------|--------|---------|---------------------|------------------------|----------------------|
| 12345   | sample_vcf_file.vcf | 1.2GB | VCF     | 2023-10-01T12:00:00Z | sequencer_device_12345  | researcher_john_doe   |

**Code:**

```python
# Example ETL Job Definition Using AWS Glue
import awsglue.transforms as T

def etl_job(glue_context, job_config):
    # Load raw data from S3
    file = glueContext.create_dynamic_frame.from_options(
        format='csv', connection_options={'paths': ['s3://genomics-data/raw/']}, format_options={'withHeader': True})

    # Transform the data (example: filtering or aggregating)
    transformed_data = T.apply_mapping(file, mappings=[("column1", "string", "column2", "int")])

    # Write processed data to a warehouse
    glueContext.write_dynamic_frame.from_options(frame=transformed_data, connection_type='s3', format='parquet', connection_options={'path': 's3://genomics-data/processed/'})
```

**Example input / output:**

```text
Input:
- BAM and VCF files stored in S3 with metadata details (filename, size, format, timestamp, provenance)

Output:
- Metadata catalog entries for each file (id, filename, size, format, timestamp, provenance)
- Processed data stored in a structured format (e.g., Parquet) in another S3 bucket
```

**Data:**

```json
{
  "dataset": {
    "filename": "sample_vcf_file.vcf",
    "size": 1234567890,
    "format": "VCF",
    "timestamp": "2023-10-01T12:00:00Z",
    "provenance": {
      "source": "sequencer_device_12345",
      "user_id": "researcher_john_doe"
    }
  },
  "metadata_index_entry": {
    "id": "md5_sample_vcf_file.vcf",
    "filename": "sample_vcf_file.vcf",
    "size": 1234567890,
    "format": "VCF",
    "timestamp": "2023-10-01T12:00:00Z",
    "provenance_source": "sequencer_device_12345",
    "provenance_user_id": "researcher_john_doe"
  }
}
```

**Flow:**

1. Store BAM/VCF Files in ObjectStorage
2. Maintain Metadata Index
3. Provide Query Capabilities viaAPI
4. Process Data Using ETL Jobs
5. Govern Data with IAM Policiesand Audit Trails

