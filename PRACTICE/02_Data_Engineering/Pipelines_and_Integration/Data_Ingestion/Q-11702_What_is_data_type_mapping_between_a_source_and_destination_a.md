---
type: "practice"
difficulty: "Intermediate"
---

# What is data type mapping between a source and destination, and what can go wrong?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Managed Ingestion and Re-syncs](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Managed_Ingestion_and_Re-syncs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Data type mapping in data ingestion involves converting data types from the source system to match those required by the target database. This process is crucial for ensuring data integrity and preventing errors, but it can also lead to issues if not handled correctly.

When working with data ingestion tools like Airbyte or Fivetran, one of the key challenges is data type mapping between the source and destination systems. The primary goal here is to ensure that the data types in your source (like CSV files, databases, or APIs) are appropriately converted into compatible formats for your target database (such as PostgreSQL, Snowflake, or BigQuery). This process can involve converting strings to dates, integers to floats, or customizing formats.

#### Triage
What is Being Probed and Assumptions

The question is probing whether you understand the concept of data type mapping in data ingestion and its importance. It also checks your ability to recognize potential pitfalls that might arise from improper type mapping. Key assumptions include:

You are familiar with common data types used by different databases.
You have an understanding of how these tools handle schema changes during incremental or full loads.

#### First Principles Solution
Step 1: Understanding Data Types

First, let's look at some common data types:

Source System: Often includes VARCHAR (string), INT (integer), DATE, TIMESTAMP, etc.
Target Database: May use more specific types like TEXT, INTEGER, DATE, DATETIME, DECIMAL, etc.
Step 2: Mapping Rules

Airbyte and Fivetran typically allow you to map these types manually or automatically. Here’s a simple example:

If your source data is stored as VARCHAR (string) in the date format "YYYY-MM-DD", it needs to be mapped correctly to the target database's DATE type.
Step 3: Handling Edge Cases

Incorrect Format: Suppose your source has dates as strings, e.g., "2023-10-05". If you try to insert this directly into a DATE column in the target, it will fail due to incorrect format.

#### Solution
Use the transformation logic within Airbyte or Fivetran to convert the string to the correct date format before insertion.

Precision Loss: When mapping floats to integers, precision loss can occur if the float value cannot be represented exactly as an integer.

#### Solution
Ensure that you understand and handle these cases properly to avoid truncation or rounding issues.

Null Handling: Proper handling of NULL values is also crucial. For instance, some databases might not allow NULL in certain columns where others might have a default value.

#### Solution
Map NULLs appropriately; for example, ensure that a NULL from the source maps correctly to a NULL or a specific default value in the target.

Complex Types: Handling complex types like arrays or JSON fields can be tricky.

#### Solution
Use the appropriate mapping configurations and possibly additional transformations to ensure data integrity.
Step 4: Best Practices
Always test your mappings thoroughly with sample data before full deployment.
Use logging mechanisms to track any issues that arise during ingestion.
Document all schema changes and type mappings for future reference and troubleshooting.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, data type mapping is essential in ensuring seamless data transfer between source and destination systems. However, it can introduce several pitfalls if not managed correctly. Some common mistakes include:

Failing to handle date formats correctly.
Not accounting for precision loss when converting types.
Overlooking NULL value handling.

For follow-ups, you might be asked about specific tools or best practices to mitigate these issues, or how to handle schema changes dynamically during incremental loads.

**Comparison:**

```
Source Field	Source Type	Target Field	Target Type
id	INTEGER	ID	BIGINT
name	VARCHAR(50)	Name	TEXT
birthdate	DATE	Birthdate	TIMESTAMP
```

**Code:**

```python
# Example Python code snippet using Airbyte API to map data types
def map_data_types(source_type, target_type):
    if source_type == "VARCHAR" and target_type == "DATE":
        return lambda x: datetime.strptime(x, "%Y-%m-%d").date()
    elif source_type == "FLOAT" and target_type == "INTEGER":
        return lambda x: int(round(float(x)))
    else:
        return lambda x: x

# Example usage
source_date_str = "2023-10-05"
target_date_obj = map_data_types("VARCHAR", "DATE")(source_date_str)
print(target_date_obj)  # Output: 2023-10-05
```

**Example input / output:**

```text
Input:
- Source: id=1, name="John Doe", birthdate="2000-01-01"
- Target Schema: ID (BIGINT), Name (TEXT), Birthdate (TIMESTAMP WITH TIME ZONE)

Output:
- id=1 -> ID=1
- name="John Doe" -> Name="John Doe"
- birthdate="2000-01-01" -> Birthdate="2000-01-01 00:00:00+00"

Explanation:
The "birthdate" value is converted from VARCHAR to TIMESTAMP, preserving the date format.
```

**Data:**

```json
{
    "source_schema": {
        "fields": [
            {"name": "id", "type": "INTEGER"},
            {"name": "name", "type": "VARCHAR(50)"},
            {"name": "birthdate", "type": "DATE"}
        ]
    },
    "target_schema": {
        "fields": [
            {"name": "ID", "type": "BIGINT"},
            {"name": "Name", "type": "TEXT"},
            {"name": "Birthdate", "type": "TIMESTAMP WITH TIME ZONE"}
        ]
    }
}
```

**Flow:**

1. Source Data
2. Map Data Types
3. Transform and Load
4. Target Database

