---
type: "practice"
difficulty: "Expert"
---

# How do you define the contract at each medallion boundary so that Bronze-to-Silver and Silver-to-Gold ownership is unambiguous across a 40-team data platform?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Medallion Layer Quality](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Medallion_Layer_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To define the contracts at each medallion boundary in a Data Engineering architecture, specifically within the context of the Bronze, Silver, and Gold layers, we need to ensure clear ownership and unambiguous boundaries across a 40-team data platform. The key is to establish well-defined interfaces that allow seamless data flow while maintaining data integrity.

#### Triaging the Question

The question essentially asks how to define the contracts (i.e., the rules or agreements) at each medallion boundary—specifically, Bronze-to-Silver and Silver-to-Gold—to ensure clear ownership and avoid ambiguity. The key points we need to address are:

Data Schema Consistency: Ensuring that data is consistently structured across layers.
Transformation Rules: Defining how data is transformed at each layer.
Metadata Management: Maintaining a robust metadata system for traceability.
Ownership and Accountability: Clearly defining who owns the transformation logic and the data at each stage.

#### First-Principles Solution

Data Schema Consistency:

Bronze Layer (Raw Data): Raw, unprocessed data ingested directly from sources. The schema should capture all raw fields.
Silver Layer (Cleaned & Processed Data): Clean and transformed data with a simplified schema that retains the essential features needed for analysis.
Gold Layer (Insightful Data): Aggregated and summarized data that is optimized for specific business metrics.

Transformation Rules:

Define clear rules for how data is cleaned, aggregated, and transformed at each layer.
Use transformation logic that can be audited and versioned to ensure traceability.
Implement a metadata management system to document the transformations applied at each stage.

Ownership and Accountability:

Assign specific teams or roles responsible for defining and maintaining the rules at each medallion boundary.
Ensure there is a clear governance framework in place that outlines the responsibilities of each team.

Metadata Management:

Maintain detailed metadata records for all data transformations, including lineage, schema evolution, and transformation logic.
Use tools like Data Catalogs or Metadata Management Platforms to manage these metadata effectively.

Testing and Validation:

Implement rigorous testing at each layer to ensure data quality before moving it to the next layer.
Use continuous integration and validation pipelines to automatically check data integrity and transformations.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes
One of the most common mistakes is not maintaining consistent schema definitions across layers, leading to data incompatibility issues. Another mistake is failing to define clear ownership and accountability for each transformation logic, which can result in uncoordinated efforts and miscommunication.

#### Follow-Up
Questions:
How would you handle versioning of transformations if multiple teams are involved?
What tools or platforms do you recommend for metadata management at scale?

**Comparison:**

| Data Layer | Schema Fields |
|------------|---------------|
| Bronze     | raw_id, timestamp, value       |
| Silver     | id, date, amount               |
| Gold       | customer_id, monthly_revenue   |

**Code:**

```python
# Example Python code to illustrate a simple transformation rule
def clean_and_aggregate(data):
    cleaned_data = data.dropna()  # Remove null values
    aggregated_data = cleaned_data.groupby(['key']).sum().reset_index()
    return aggregated_data

# Example metadata dictionary
metadata = {
    "source_table": "raw_sales_data",
    "target_table": "cleaned_sales_aggregates",
    "transformation_logic": clean_and_aggregate.__doc__,
    "version": 1.0
}
```

**Example input / output:**

```text
Input (Bronze Layer):
raw_id | timestamp         | value
-------|------------------|------
1      | 2023-01-01 08:00  | 10.5
2      | 2023-01-01 09:00  | 5.3

Output (Silver Layer):
id     | date             | amount
-------|------------------|---------
1      | 2023-01-01       | 7.9

Input (Silver Layer):
id     | date             | amount
-------|------------------|---------
1      | 2023-01-01       | 7.9
2      | 2023-01-01       | 5.3

Output (Gold Layer):
customer_id | monthly_revenue
-------------|-----------------
1           | 13.2
```

**Data:**

```json
{
  "bronze_layer_schema": {
    "fields": [
      {"name": "raw_id", "type": "string"},
      {"name": "timestamp", "type": "datetime"},
      {"name": "value", "type": "float"}
    ]
  },
  "silver_layer_schema": {
    "fields": [
      {"name": "id", "type": "string"},
      {"name": "date", "type": "date"},
      {"name": "amount", "type": "float"}
    ]
  },
  "gold_layer_schema": {
    "fields": [
      {"name": "customer_id", "type": "string"},
      {"name": "monthly_revenue", "type": "float"}
    ]
  }
}
```

**Flow:**

1. Bronze Layer
2. Clean & Aggregate
3. Silver Layer
4. Aggregate & Summarize
5. Gold Layer

