---
type: "practice"
difficulty: "Basic"
---

# What is DBFS (Databricks File System) and how is it related to underlying cloud storage like S3 or ADLS?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Databricks Ingestion](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Ingestion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

DBFS, or Databricks File System, is a high-performance file storage system designed by Databricks to enable seamless access to data across various cluster types and applications within a Databricks environment. It abstracts away the complexities of managing distributed file systems, allowing users to focus on analytics rather than infrastructure.

To understand DBFS in relation to underlying cloud storage like S3 or ADLS (Azure Data Lake Storage), we need to consider its key features, advantages, and how it integrates with these storage services.

**Comparison:**

| Feature       | DBFS                             | S3                                     | ADLS                                    |
|---------------|----------------------------------|-----------------------------------------|------------------------------------------|
| Purpose       | High-performance file system for Databricks  | Object storage service                 | Unified data lake storage solution        |
| Integration   | Direct access                    | API-based interactions                  | Secure and scalable                      |
| Performance    | Optimized for read operations     | Standard performance                   | Enhanced performance                     |

**Example input / output:**

```text
Input: A file is uploaded to an S3 bucket named 'my-bucket' with the path 'data.csv'.

Output:
1. The file 'data.csv' is accessible via DBFS using the same path: dbfs:/mnt/my-bucket/data.csv.
2. Databricks clusters can read and write this data directly, leveraging optimized caching for faster processing.
3. Operations like listing files or reading contents are performed seamlessly, abstracting away the underlying storage details.
```

**Data:**

```json
{
  "DBFS": {
    "description": "A high-performance file system optimized for Databricks workloads",
    "features": [
      "Integrated caching and read performance optimizations",
      "Seamless access to S3, ADLS, and other cloud storage services",
      "Supports POSIX-compliant APIs"
    ],
    "integration_with_cloud_storage": {
      "S3": "Facilitates direct interaction with AWS S3 buckets",
      "ADLS": "Enables connectivity with Azure Data Lake Storage accounts"
    }
  },
  "Cloud_Storage_Services": {
    "S3": "Amazon Simple Storage Service, a scalable object storage service",
    "ADLS": "Azure Data Lake Storage, a unified data lake storage solution"
  }
}
```

**Flow:**

1. Yes
2. No
3. File accessible viadbfs:/mnt/my-bucket/data.csv
4. User uploads file to S3
5. Is DBFS enabled?
6. DBFS abstracts the path andprovides a uniform interface
7. Direct interaction with S3 orADLS
8. Cluster reads data from DBFS
9. Data is processed withinDatabricks environment
10. Data is read directly from thecloud storage service

