---
type: "practice"
difficulty: "Deep"
---

# A data product needs to join a table in BigQuery with one in Snowflake for a single report — what's the least-bad architecture, and why is 'just federate it' usually the wrong default?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-cloud Data Contracts](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of joining tables from BigQuery and Snowflake for a single report, we must carefully consider the architectural trade-offs involved in multi-cloud data integration. The primary goal is to achieve seamless data access while minimizing costs, operational complexity, and performance overhead. The phrase "just federate it" often suggests a simplistic approach that may overlook critical factors such as data consistency, cost optimization, and governance.

In this context, we should explore an architecture that leverages both cloud platforms' strengths. A common and effective strategy is to replicate the necessary data from Snowflake to BigQuery or create a unified data lake solution using a third-party service like Amazon S3. This approach balances performance, cost, and manageability. Let's break down why 'just federate it' is often inadequate.

**Code:**

```

graph LR; A[BigQuery] --> B[Replication Layer]; C[Snowflake] --> B; B --> D[Unified Data Lake]; E[Data Report] --> D;
```

**Example input / output:**

```text
No specific input-output examples are needed, but a dry-run of the process could be as follows:

1. **Extract Data from Snowflake**: Use AWS Glue or a similar service to extract data from Snowflake and load it into S3.
2. **Transform and Load to BigQuery**: Apply any necessary transformations in AWS Glue or another ETL tool, then write the data to BigQuery.
3. **Join Tables for Reporting**: Query both BigQuery and the transformed data directly.
```

**Data:**

```

No specific data schemas are required for this conceptual explanation, but a typical setup might involve replication from Snowflake tables to BigQuery.

YAML
Copy
# Example BigQuery Table Schema
snowflake_table:
  - id: INT64
  - name: STRING
  - value: NUMERIC

bigquery_table:
  - id: INT64
  - name: STRING
  - value: NUMERIC
```

**Flow:**

1. Extract
2. S3
3. Transform
4. Load BigQuery
5. BigQuery Table
6. Join with Snowflake Data
7. Unified Reporting

