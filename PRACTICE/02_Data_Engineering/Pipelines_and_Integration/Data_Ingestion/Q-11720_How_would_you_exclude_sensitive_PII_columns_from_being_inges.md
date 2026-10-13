---
type: "practice"
difficulty: "Deep"
---

# How would you exclude sensitive PII columns from being ingested into the warehouse?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Ingestion Credentials](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Ingestion_Credentials.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To exclude sensitive Personal Identifiable Information (PII) columns during data ingestion, especially when using tools like Airbyte or Fivetran, we need to ensure that only non-sensitive data reaches our warehouse. This involves understanding what PII means in your context, configuring the ETL tool appropriately, and possibly implementing additional checks within your pipeline.

The question is probing on how you handle sensitive information during the data ingestion process. We will assume some common scenarios: you are using Airbyte or Fivetran for ingestion, and your goal is to exclude certain columns that contain PII from being ingested into the warehouse.

To achieve this, we need to follow these steps:

Identify Sensitive Columns: Define which columns contain sensitive information.
Configure ETL Tool Settings: Configure Airbyte or Fivetran settings to exclude specific columns during ingestion.
Implement Additional Checks: Optionally, add checks in your pipeline to ensure that even if the data makes it past Airbyte/Fivetran, it is still protected.

Let's break this down step-by-step from first principles.

Step 1: Identify Sensitive Columns

First, identify which columns contain sensitive information such as names, social security numbers (SSNs), credit card details, etc. This identification might be done through a data discovery process or by consulting with stakeholders who know the data schema and its intended uses.

Step 2: Configure ETL Tool Settings

Once we have identified the PII columns, we need to configure Airbyte or Fivetran to exclude these columns during ingestion:

Airbyte: You can use a schema override approach where you define which fields should be excluded in your connector configuration. For example:

YAML
destination_sync_mode: append
schema_overrides:
  - table_name: customer_data
    column_blacklist:
      - sensitive_column1
      - sensitive_column2

Fivetran: Fivetran also supports blacklisting columns in its connector configurations. You can specify which fields should be excluded during the data ingestion process.

Step 3: Implement Additional Checks

While Airbyte and Fivetran help significantly, it’s a good practice to have additional checks within your pipeline to ensure that sensitive information is not ingested even if somehow bypassed by the ETL tool. This can include:

Data Masking: Use data masking techniques in your ETL process to obfuscate PII before it reaches the warehouse.
Data Transformation: Transform sensitive columns on-the-fly using a transformation layer in your pipeline, ensuring that only non-sensitive values are stored.
Step 4: Monitoring and Auditing

Finally, ensure you have proper monitoring and auditing in place to detect any potential issues or breaches. This might include setting up alerts for unusual data patterns and regularly reviewing logs.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By following these steps, we can effectively exclude sensitive PII columns from being ingested into the warehouse using Airbyte or Fivetran. The key is thorough identification of sensitive data, proper configuration in ETL tools, and additional layers of protection within the pipeline.

#### Common mistakes
include:

Not identifying all sensitive columns accurately.
Overlooking the need for additional checks beyond the ETL tool.
Failing to implement robust monitoring and auditing mechanisms.

For follow-ups, we might discuss more advanced techniques like data anonymization or encryption, as well as how to handle different types of PII in various jurisdictions.

**Code:**

```yaml
# Example Airbyte Schema Override Configuration
destination_sync_mode: append
schema_overrides:
  - table_name: customer_data
    column_blacklist:
      - sensitive_column1
      - sensitive_column2
```

