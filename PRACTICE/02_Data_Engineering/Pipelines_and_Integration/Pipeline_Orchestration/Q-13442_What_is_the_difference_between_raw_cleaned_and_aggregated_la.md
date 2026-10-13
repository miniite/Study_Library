---
type: "practice"
difficulty: "Deep"
---

# What is the difference between raw, cleaned, and aggregated layers (bronze silver gold) in a telemetry pipeline?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Feature Pipelines](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/GDPR_Deletion_in_Feature_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AIOps environment, data pipelines often follow a "trifecta" model of processing raw, cleaned, and aggregated data, labeled as bronze, silver, and gold layers respectively. This approach ensures that each layer serves a specific purpose in the pipeline's lifecycle, from initial ingestion to final consumption.

#### Triaging the Question

The core of this question is understanding the differences between these three stages: raw (bronze), cleaned (silver), and aggregated (gold). The interviewer likely wants you to explain how each stage processes data, why they are important, and what problems they solve.

#### Assumptions
We assume a typical telemetry pipeline that collects metrics or logs from various sources.
Raw data is typically unprocessed and contains all the raw information collected.
Cleaned (silver) data has been processed to ensure consistency and reliability.
Aggregated (gold) data consolidates cleaned data into higher-level summaries for analysis.

#### First Principles Solution
Raw Layer (Bronze)

The raw layer, or bronze stage, deals with ingesting raw telemetry data from various sources such as IoT devices, servers, sensors, etc. This raw data often comes in a raw format without any processing:

PLAINTEXT
Raw Data Example:
{
    "timestamp": "2023-10-01T14:00:00Z",
    "device_id": 12345,
    "temperature": -40.0,  // Could be incorrect due to sensor failure
    "humidity": 99.0       // Outlier value
}

At this stage:

Data is ingested directly without any transformation.
Ensures nothing is lost during the initial collection phase.
Cleaned Layer (Silver)

The cleaned layer, or silver stage, focuses on ensuring data quality and consistency by cleaning the raw data. This includes tasks such as removing outliers, handling missing values, and converting data types appropriately:

PLAINTEXT
Cleaned Data Example:
{
    "timestamp": 1633094400000,
    "device_id": 12345,
    "temperature": -45.0,  // Corrected from -40 due to known sensor issue
    "humidity": 75.0       // Normalized from outlier value
}

At this stage:

Data is transformed and cleaned.
Ensures reliability by handling edge cases like outliers.
Aggregated Layer (Gold)

The aggregated layer, or gold stage, consolidates the cleaned data into useful metrics for analysis. This involves aggregating data over time windows, calculating statistics, and preparing it for consumption by business intelligence tools:

PLAINTEXT
Aggregated Data Example:
{
    "date": "2023-10-01",
    "device_id": 12345,
    "avg_temperature": -47.5,
    "min_temperature": -50.0,
    "max_temperature": -45.0
}

At this stage:

Data is summarized and aggregated.
Provides high-level insights for decision-making.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the trifecta model ensures that data flows through different stages of processing to maintain quality, reliability, and usability. The raw layer captures initial data without alteration; cleaned data addresses inconsistencies and errors; and aggregated data provides high-level summaries for analysis.

#### Common mistakes
include:

Skipping cleaning steps, leading to unreliable downstream processes.
Over-aggregation or under-aggregation, which can skew analysis results.

For follow-ups, one might ask about specific tools used (e.g., Airflow/Spark), how these layers integrate with other systems, and potential challenges in managing large-scale telemetry data pipelines.

**Comparison:**

| Layer   | Description                                       | Example Output                |
|---------|---------------------------------------------------|-------------------------------|
| Bronze  | Raw, unprocessed data                            | {"timestamp": ..., "temperature": ..., "humidity": ...} |
| Silver  | Cleaned and consistent data                      | {"timestamp": ..., "temperature": ..., "humidity": ...} |
| Gold    | Aggregated summary for analysis                   | {"date": "2023-10-01", "avg_temperature": ..., "min_temperature": ..., "max_temperature": ...} |

**Code:**

```python
# Example of a simple transformation in Python using Spark for cleaning
from pyspark.sql import functions as F

raw_data = spark.read.json("path/to/raw_data.json")

# Cleaning example
cleaned_data = raw_data \
    .withColumn("timestamp", F.from_unixtime(F.col("timestamp"), "yyyy-MM-dd'T'HH:mm:ssZ")) \
    .withColumn("temperature", F.when(F.abs(F.col("temperature")) > 50, -45.0).otherwise(F.col("temperature"))) \
    .withColumn("humidity", F.when(F.abs(F.col("humidity") - 100) < 25, (F.col("humidity") + 25) / 2).otherwise(75.0))
```

**Example input / output:**

```text
Input (Raw Data):
{
    "timestamp": "2023-10-01T14:00:00Z",
    "device_id": 12345,
    "temperature": -40.0,  // Raw data
    "humidity": 99.0       // Outlier value
}

Output (Cleaned Data):
{
    "timestamp": 1633094400000,
    "device_id": 12345,
    "temperature": -45.0,  // Corrected temperature
    "humidity": 75.0       // Normalized humidity
}

Output (Aggregated Data):
{
    "date": "2023-10-01",
    "device_id": 12345,
    "avg_temperature": -47.5,
    "min_temperature": -50.0,
    "max_temperature": -45.0
}
```

**Data:**

```

{ "timestamp": "2023-10-01T14:00:00Z", "device_id": 12345, "temperature": -40.0, // Raw data "humidity": 99.0 // Outlier value }

CODE
Copy

```json
{
    "timestamp": 1633094400000,
    "device_id": 12345,
    "temperature": -45.0,  // Corrected temperature
    "humidity": 75.0       // Normalized humidity
}
JSON
Copy
{
    "date": "2023-10-01",
    "device_id": 12345,
    "avg_temperature": -47.5,
    "min_temperature": -50.0,
    "max_temperature": -45.0
}
```

**Flow:**

1. Raw Data Ingestion
2. Cleaning and Transformation
3. Aggregation and Summary
4. Analysis
5. Data Quality Checks
6. Error Handling

