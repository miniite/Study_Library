---
type: "practice"
difficulty: "Intermediate"
---

# How would you clean a JSON column in a warehouse by extracting nested fields into flat columns?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Reject vs Quarantine Bad Rows](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, cleaning and transforming JSON columns is a common task to ensure data consistency and usability. For this problem, we'll focus on extracting nested fields from a JSON column in a warehouse and converting them into flat columns. The core idea here is to use SQL queries or ETL (Extract, Transform, Load) tools to flatten the JSON structure while preserving relevant information.

The question is essentially asking how to process JSON data stored in a database column and convert it into a more structured format that can be queried easily. We need to make some assumptions: first, we have access to an SQL-based warehouse or a data processing tool like Apache Spark; second, the JSON structure might vary across rows, but we'll focus on common fields.

Let's go through the solution step by step:

#### Assumptions
and Context: Suppose you are working with a table called orders that has a column named details, which contains nested JSON objects like this:

JSON
{"customer_id": 123, "order_id": 456, "items": [{"name": "Laptop", "price": 999}, {"name": "Mouse", "price": 20}]}

#### First-Principles Solution

We need to parse the JSON and extract key-value pairs.
For nested fields, we'll flatten them into separate columns.

Step-by-Step Breakdown:

Extract Simple Fields: Use SQL functions like JSON_EXTRACT (for MySQL) or json_value (in SQL Server).
Handle Nested Fields: Use recursive queries or specific JSON processing functions to extract and flatten nested data.
Normalization: Ensure that the extracted fields are appropriately normalized.

SQL Example:

SQL
-- Sample SQL query using PostgreSQL's jsonb data type
SELECT
    order_id,
    customer_id,
    jsonb_array_elements(items)->>'name' AS item_name,
    jsonb_array_elements(items)->>'price'::float AS item_price
FROM orders,
     jsonb_array_elements(details->'items') AS items;

ETL Tool Example:

Using Apache Spark with PySpark:
PYTHON
from pyspark.sql.functions import col, explode

# Load data
df = spark.read.json("/path/to/orders.json")

# Explode nested array and flatten fields
flat_df = df.withColumn("items", explode(col("details.items")))
            .select(
                "order_id",
                "customer_id",
                col("items.name").alias("item_name"),
                col("items.price").cast("float").alias("item_price")
            )

# Show result
flat_df.show()

#### Closing Summary
and Common Mistakes:

Ensure that the extracted fields are correctly typed (e.g., integers, floats).
Handle edge cases where JSON might be missing or malformed.
Verify data quality after transformation.

