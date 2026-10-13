---
type: "practice"
difficulty: "Deep"
---

# How would you build a metadata/config-driven ingestion framework in Databricks so onboarding a new source table is a config change, not new code?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Databricks Ingestion](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Ingestion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a metadata/config-driven ingestion framework in Databricks, we need to ensure that adding or changing data sources can be done through configuration changes rather than writing and deploying new code. This approach will make our system more maintainable and scalable, especially as the number of data sources grows.

#### Triaging the Question

The core challenge is to decouple data ingestion logic from the underlying codebase by leveraging Databricks’ capabilities in metadata management and automation. We'll need to design a solution that allows us to define data sources, transformation rules, and other necessary configurations in a declarative manner. The key is to ensure these configurations are easily modifiable and can be applied without requiring developers to write new code.

#### Assumptions
Data Sources: We will have various types of data sources (e.g., S3 buckets, databases, APIs).
Transformation Rules: These rules specify how to clean, transform, and load the data.
Metadata Management: Using Databricks metadata to store and retrieve configurations.

#### First-Principles Solution
Define Data Sources: Use a metadata store (e.g., Delta Lake) or DBFS to define each data source as a configuration entry.
Transformation Rules: Implement these rules using SQL, PySpark, or other Databricks-supported languages and store them in the same metadata store.
Automation Framework: Create a framework that reads configurations from the metadata store and dynamically loads corresponding ingestion logic.
Step-by-Step Solution

Metadata Schema Design

Define a schema to store data source details (e.g., location, format).
Define another schema for transformation rules.

Metadata Store Implementation

Use Delta Lake or DBFS to persist the configuration metadata.
PYTHON
from delta import DeltaTable

# Example: Create a Delta table for storing data sources
DeltaTable.createOrReplace("sources").registerTempTable("sources")

Transformation Rules Management

Store transformation rules as code snippets in DBFS or within the same Delta Lake.
PYTHON
# Example: Store a simple PySpark transformation rule
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

def transform_data(df):
    return df.filter(df.column1 > 0).select("column2", "column3")

# Register the function as a UDF or save it to DBFS for dynamic loading

Automation Framework

Develop an automation framework that dynamically loads configurations and applies them.
PYTHON
from delta.tables import DeltaTable

def ingest_data():
    # Load data sources metadata
    source_df = DeltaTable.forName(spark, "sources").toDF()

    for row in source_df.collect():
        data_source_info = {
            "location": row.location,
            "format": row.format
        }

        # Load corresponding transformation rule
        rule_path = f"/dbfs/{row.rule_path}"
        with open(rule_path, 'r') as file:
            rule_code = file.read()

        exec(rule_code)

        # Apply the rule to the data source and write the result
        df = spark.read.format(row.format).load(row.location)
        transformed_df = transform_data(df)
        transformed_df.write.mode("overwrite").saveAsTable(f"{row.table_name}")

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we've outlined a metadata-driven approach to data ingestion in Databricks by leveraging Delta Lake and PySpark for managing configurations. This ensures that onboarding new sources is straightforward through configuration changes rather than code modifications.

#### Common Mistakes

Not using version control for the metadata store.
Hardcoding paths or values instead of dynamically loading them.
Failing to test transformation rules thoroughly before applying them in production.

#### Follow-Ups

How would you ensure data quality and validation as part of this framework?
What are some potential scalability challenges, and how can they be addressed?

**Comparison:**

| Source | Format | Rule Path |
|--------|--------|-----------|
| s3://example-bucket/data | parquet | /dbfs/rules/transform_rule.py |
| jdbc:mysql://localhost:3306/mydatabase | csv | /dbfs/rules/csv_transform.py |

**Code:**

```python
# Example code blocks for demonstration purposes
from delta.tables import DeltaTable

def ingest_data():
    source_df = DeltaTable.forName(spark, "sources").toDF()
    for row in source_df.collect():
        data_source_info = {
            "location": row.location,
            "format": row.format
        }
        rule_path = f"/dbfs/{row.rule_path}"
        with open(rule_path, 'r') as file:
            rule_code = file.read()

        exec(rule_code)

        df = spark.read.format(row.format).load(row.location)
        transformed_df = transform_data(df)
        transformed_df.write.mode("overwrite").saveAsTable(f"{row.table_name}")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| A new source table with configuration added to `sources` Delta Lake. | Data ingestion logic automatically runs, applying defined rules and writing results to the specified tables. |

**Data:**

```json
{
  "sources": [
    {
      "location": "s3://example-bucket/data",
      "format": "parquet",
      "rule_path": "/dbfs/rules/transform_rule.py"
    },
    {
      "location": "jdbc:mysql://localhost:3306/mydatabase",
      "format": "csv",
      "rule_path": "/dbfs/rules/csv_transform.py"
    }
  ]
}
```

**Flow:**

1. Define Data Sources
2. Store in Delta Lake
3. Define Transformation Rules
4. Store in DBFS
5. Dynamically LoadConfigurations
6. Apply Rule and Ingest Data
7. Write Result to Table

