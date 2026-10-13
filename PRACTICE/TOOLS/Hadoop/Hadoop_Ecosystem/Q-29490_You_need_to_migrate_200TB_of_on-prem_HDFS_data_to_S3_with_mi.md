---
type: "practice"
difficulty: "Expert"
---

# You need to migrate 200TB of on-prem HDFS data to S3 with minimal downtime. How would you plan the migration, validate integrity, and cut over pipelines?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ HDFS Data Integrity](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/HDFS_Data_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate 200TB of on-prem HDFS data to S3 with minimal downtime, we need a carefully planned strategy that includes parallel processing, error handling, and seamless cutover. The key steps are:

Preparation: Assess the current environment, set up AWS services, and prepare the target S3 buckets.
Data Migration: Use Hadoop tools to transfer data in parallel while ensuring integrity checks.
Validation: Verify the correctness of the migrated data on both HDFS and S3.
Cutover Pipelines: Safely switch over pipelines from HDFS to S3.

Let's break this down step-by-step:

Preparation
Assess Current Environment:
Understand the current HDFS configuration, including block size, replication factor, and any existing data access patterns.
Set Up AWS Services:
Create S3 buckets with appropriate naming conventions for versioning if necessary.
Configure VPCs and security groups to enable secure data transfer between on-premises and AWS.
Prepare Data Migration Tools:
Use Hadoop tools like aws s3 cp or AWS DataSync, which support large-scale data migrations.
Data Migration
Parallel Processing:
Split the data into smaller chunks (e.g., 10TB each) and use parallelism to speed up the migration.
Employ multi-threaded Hadoop jobs with mapred-stream or custom scripts for efficient data transfer.
Data Transfer:
Utilize AWS DataSync, a managed service that can handle terabytes of data in hours.
For large datasets, consider using Hadoop with tools like aws s3 cp to copy data in batches.
Validation
Checksum Verification:
Calculate checksums (e.g., MD5 or SHA-256) on both the source and target sides.
Compare these checksums after migration to ensure data integrity.
Data Consistency Checks:
Run Hadoop MapReduce jobs to verify file contents match between HDFS and S3.
Use AWS Lambda functions to trigger these checks if there's a change in metadata or files.
Cutover Pipelines
Pilot Migration:
Perform a small-scale migration first to test the entire process, including error handling and validation steps.
Final Migration:
Once validation is successful, proceed with the full migration, monitoring all stages closely.
Switch Over:
Safely switch over pipelines by updating configuration files to point to S3 instead of HDFS.
Use golden records or ETL processes to ensure real-time data availability.

#### Closing Summary

This strategy ensures a smooth and reliable transition from on-premises HDFS to AWS S3, minimizing downtime and ensuring data integrity. Key steps include thorough preparation, parallelized data migration, rigorous validation checks, and a structured cutover plan.

#### Common Mistakes/Follow-Ups
Overlooking Integrity Checks: Failing to perform checksums or consistency checks can lead to undetected data corruption.
Neglecting Security: Poorly configured VPCs or security groups can expose sensitive data during the migration.
Insufficient Monitoring: Without proper monitoring, issues like network latency or partial failures may go unnoticed.

**Comparison:**

```
Step	Action
1	Assess current environment
2	Set up AWS services
3	Prepare Hadoop tools
4	Parallel data transfer
5	Validate integrity
6	Cutover pipelines
```

**Code:**

```
SH
Copy
# Example Hadoop command for checksum verification
hdfs dfs -checksum /path/to/source/directory | aws s3 cp - s3://bucket/checksums/
```

**Example input / output:**

```text
Input:
- On-prem HDFS with 200TB of data
- Target S3 buckets

Output:
- Migrated and validated data in S3
- Updated pipelines to use new S3 location
```

**Data:**

```json
{
  "HDFS_Bucket": "/path/to/data",
  "S3_Buckets": [
    {
      "Name": "data-migration-bucket",
      "VersioningEnabled": true,
      "LoggingEnabled": false
    }
  ]
}
```

**Flow:**

1. Assessment
2. Set up AWS Services
3. Prepare Hadoop Tools
4. Parallel Data Transfer
5. Validation Checksums
6. Run Consistency Checks
7. Safely Switch Over Pipelines
8. Migrate and Validate Data

