---
type: "practice"
difficulty: "Expert"
---

# How would you architect MDM to feed a real-time personalization/decisioning engine where a stale identity means a bad recommendation — what's the freshness contract and fallback?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ MDM for Batch and Streaming](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/MDM_for_Batch_and_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect Master Data Management (MDM) for feeding a real-time personalization/decisioning engine, we need to ensure that identities are fresh and reliable at all times. The key challenge here is defining a freshness contract—how recent the data needs to be—and establishing fallback mechanisms in case of identity staleness. Let's break this down from first principles.

Firstly, the MDM system must integrate with various sources (e.g., CRM systems, transactional databases) and maintain a single source of truth for customer identities. We'll use Apache Kafka for real-time data ingestion, Amazon Redshift for bulk processing, and AWS Glue for ETL jobs to sync multiple data sources.

The freshness contract defines the acceptable age limit for identity information before it becomes stale. For instance, if we require near-real-time recommendations, a 15-minute freshness threshold might be appropriate. We'll use this threshold to trigger reprocessing of data when identities become stale, ensuring that our decisioning engine always has up-to-date customer profiles.

Now let's dive into the architectural details and reasoning step-by-step.

**Comparison:**

```
Data Source	Frequency of Updates
CRM	Real-time
Transaction Database	Every 15 minutes
Freshness Contract
Customer data must be less than 15 minutes old to be considered fresh.
```

**Code:**

```python
# Example Python script for ETL using AWS Glue
import awsglue.context as glue_context
from awsglue.dynamicframe import DynamicFrame

def main():
    # Initialize the Glue context
    glueContext = GlueContext(SparkSession)

    # Read data from multiple sources (e.g., S3, RDS)
    customer_data = glueContext.read.format("csv").option("header", "true").load("s3://customer-data/source1")
    transaction_data = glueContext.read.format("json").option("multiline", "true").load("s3://transaction-data/source2")

    # Join data and write to Redshift for bulk processing
    joined_data = customer_data.join(transaction_data, "customer_id")
    redshift_output = glueContext.write_dynamic_frame.from_jdbc_conf(frame=joined_data, job書名="mdm_to_redshift", database="mdmdb", table_name="merged_customer_transactions")

if __name__ == "__main__":
    main()
```

**Example input / output:**

```text
Input:
- Customer data from CRM (e.g., CSV)
- Transactional data from transactional database (e.g., JSON)

Output:
- Unified customer profile in Redshift

Dry-run example:
1. Ingest raw customer and transaction data into Kafka topics.
2. ETL using AWS Glue to join data, enrich profiles with recent transactions.
3. Write processed data to Amazon Redshift for analytics.
```

**Data:**

```json
{
  "customer_id": "12345",
  "first_name": "John",
  "last_name": "Doe",
  "email": "johndoe@example.com",
  "registration_date": "2023-09-01"
}
```

**Flow:**

1. Yes
2. No
3. Ingest Data
4. Is Data Fresh?
5. Update Redshift
6. Reprocess ETL Job
7. Bulk Process Data in Redshift
8. Retry ETL with Fresh Data

