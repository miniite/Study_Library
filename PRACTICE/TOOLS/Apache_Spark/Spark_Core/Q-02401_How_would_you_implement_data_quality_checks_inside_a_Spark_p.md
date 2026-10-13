---
type: "practice"
difficulty: "Deep"
---

# How would you implement data quality checks inside a Spark pipeline so bad batches fail fast rather than corrupt the warehouse?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Spark Execution Caching and Debugging](<../../../../TOOLS/Apache_Spark/Spark_Core/Spark_Execution_Caching_and_Debugging.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Implementing data quality checks in Apache Spark pipelines to ensure bad batches fail fast requires a combination of schema validation, record-level checks, and leveraging Spark's transformations and actions. The key is to use Spark’s built-in APIs for handling data validation and error detection.

To achieve this, we need to understand the problem at hand: ensuring that only valid, clean data is fed into downstream processes or stored in the data warehouse. We will focus on schema validation, record-level checks, and using Spark transformations and actions to handle failures gracefully.

#### Triaging the Question
The core of the question revolves around how to ensure data quality within a Spark pipeline such that bad batches fail fast rather than corrupting the warehouse. This involves understanding the data being processed, defining what constitutes "bad" data, and implementing checks at various stages of the pipeline.

Key Assumptions:

We are working with structured data (e.g., JSON, Parquet).
The data is read from a source (e.g., HDFS, S3) and written to a warehouse.
We have control over the schema definitions and can define validation rules.
The goal is to fail fast on bad batches without impacting downstream processes.

Step-by-Step Solution:

Schema Validation: Define the expected schema for the data using Spark's DataFrame or Dataset APIs. Use schema to validate incoming records against this predefined schema.

PYTHON
from pyspark.sql import SparkSession, Row

spark = SparkSession.builder.getOrCreate()

# Define the expected schema
expected_schema = spark.sparkContext.parallelize([Row(id=int(), name=str(), age=int())])

# Read data and validate against schema
def validate_schema(df):
    return df.schema == expected_schema

Record-Level Checks: Implement specific checks at the record level to ensure that each row conforms to business logic or quality standards.

PYTHON
from pyspark.sql.functions import col

# Example check: Ensure age is between 0 and 150
def check_age(df):
    return df.filter(col("age").between(0, 150))

# Example check: Ensure name is not null or empty
def check_name(df):
    return df.filter(col("name").isNotNull() & col("name").isNonEmptyString())

Combining Checks: Combine these checks into a single pipeline to ensure all records pass all validations.

PYTHON
from pyspark.sql import DataFrame

def validate_data(df: DataFrame) -> DataFrame:
    # Validate schema first
    if not validate_schema(df):
        raise ValueError("Schema validation failed")

    # Apply record-level checks
    df = check_age(df)
    df = check_name(df)

    return df

Handling Failures: Use Spark’s error handling mechanisms to fail fast when a batch is deemed bad.

PYTHON
try:
    validated_df = validate_data(raw_df)
except ValueError as e:
    print(f"Validation failed: {e}")
    # Optionally, log the failure or take further action
    return

# Proceed with processing only if validation succeeds
processed_df = process(validated_df)

Storing Valid Data: Ensure that only validated data is stored in the warehouse.

PYTHON
def store_data(df: DataFrame):
    df.write.format("parquet").mode("overwrite").save("warehouse/path")

Monitoring and Logging: Implement monitoring and logging to track validation results and any failures.

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

def log_validation_results(df: DataFrame):
    # Log the number of valid records
    num_valid_records = df.count()
    print(f"Validated {num_valid_records} records.")

    # Optionally, write validation results to a metrics store
    metrics_df = spark.createDataFrame([Row(num_valid_records=num_valid_records)])
    metrics_df.write.format("console").mode("append").save()

#### Closing Summary
By implementing schema validation and record-level checks within the Spark pipeline, we can ensure that only valid data is processed or stored. This approach leverages Spark’s powerful APIs to handle complex data transformations and validations efficiently. Proper error handling ensures that bad batches are detected early, preventing potential corruption of downstream processes.

#### Common Mistakes/Follow-Ups

Failing to define a clear validation schema.
Overlooking edge cases in record-level checks (e.g., null values).
Not logging or monitoring validation results.
Ignoring the need for robust error handling and retry mechanisms.
Failing to integrate with existing data governance tools.

**Comparison:**

```
Record	Validation Status
1	Valid
2	Invalid
3	Invalid
```

**Code:**

```python
from pyspark.sql import SparkSession, Row, DataFrame

spark = SparkSession.builder.getOrCreate()

# Define expected schema
expected_schema = spark.sparkContext.parallelize([Row(id=int(), name=str(), age=int())])

def validate_schema(df):
    return df.schema == expected_schema

def check_age(df):
    return df.filter(col("age").between(0, 150))

def check_name(df):
    return df.filter(col("name").isNotNull() & col("name").isNonEmptyString())

def validate_data(df: DataFrame) -> DataFrame:
    if not validate_schema(df):
        raise ValueError("Schema validation failed")

    df = check_age(df)
    df = check_name(df)

    return df

try:
    validated_df = validate_data(raw_df)
except ValueError as e:
    print(f"Validation failed: {e}")
    # Optionally, log the failure or take further action
else:
    processed_df = process(validated_df)
    store_data(processed_df)
```

**Example input / output:**

```

Input: [ {"id": 1, "name": "John Doe", "age": 25}, {"id": 2, "name": "", "age": 30}, {"id": 3, "name": "Jane Smith", "age": -1} ]

Output:

CODE
Copy
Validation failed: Schema validation failed
Processed records: 1
```

**Data:**

```json
[
  {"id": 1, "name": "John Doe", "age": 25},
  {"id": 2, "name": "", "age": 30}, // Invalid record: empty name
  {"id": 3, "name": "Jane Smith", "age": -1} // Invalid record: negative age
]
```

**Flow:**

1. Yes
2. No
3. Read Data
4. Validate Schema
5. Check Age
6. Check Name
7. Process Valid Data
8. Validation Failed?
9. Log Failure

