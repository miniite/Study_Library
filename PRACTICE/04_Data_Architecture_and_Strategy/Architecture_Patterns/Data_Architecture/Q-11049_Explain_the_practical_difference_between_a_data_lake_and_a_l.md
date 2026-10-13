---
type: "practice"
difficulty: "Basic"
---

# Explain the practical difference between a data lake and a lakehouse to a backend engineer who thinks 'we already dump JSON in S3, so we have a lake.' What are we actually missing without a table format?

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Lake Storage](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When discussing data architectures with a backend engineer, it's crucial to clarify that while dumping JSON into an Amazon S3 bucket may seem like a simple solution for storing raw data, it doesn't capture the full potential of a data lake or a lakehouse. A data lake is great for storing large volumes of raw data without predefined schemas, but it lacks structured querying capabilities and governance mechanisms. In contrast, a lakehouse leverages these benefits while adding robust schema enforcement and advanced analytics.

#### Triaging the Question

The question revolves around understanding the practical differences between a data lake (typically represented by S3 buckets) and a lakehouse (often implemented using Delta Lake or similar technologies). The underlying assumption is that dumping JSON into an S3 bucket constitutes a complete data lake solution, which is misleading. We need to highlight the essential components missing from this approach.

#### First-Principles Solution

Data Lake vs. Lakehouse:

Data Lake: Stores raw, unstructured data in its original format (e.g., JSON). It is ideal for storing vast amounts of data without upfront schema definition.
Lakehouse: Combines the flexibility of a data lake with structured query capabilities through a table format and additional governance tools.

Key Components Missing:

Schema Enforcement: Without predefined schemas, querying raw data can be cumbersome and error-prone.
Structured Querying: Tools like SQL or specific frameworks (e.g., Delta Lake) enable efficient querying of semi-structured data, making it easier to perform complex analytics.
Governance and Lifecycle Management: Proper data management practices such as retention policies, security controls, and lineage tracking are essential for compliance and operational efficiency.

#### Example Scenario

Suppose you need to analyze customer purchase behavior over time. In a traditional data lake setup, this would involve manually parsing JSON files each time, which can be slow and error-prone.
In a lakehouse like Delta Lake, you can define a schema for the purchase events, enabling efficient querying directly on the stored data.

#### Closing Summary

While dumping JSON in S3 might seem like a simple way to store raw data, it fails to provide the structured query capabilities and governance mechanisms needed for advanced analytics and operational efficiency. A lakehouse architecture addresses these gaps by combining the flexibility of a data lake with robust schema enforcement and advanced querying tools.

#### Common Mistakes/Follow-Ups
Misunderstanding Schema Flexibility: Assuming that the lack of predefined schemas means no need for any structure.
Overlooking Query Performance: Ignoring the fact that structured data allows for faster, more efficient querying.
Neglecting Governance and Lifecycle Management: Failing to recognize the importance of proper data management practices.

#### Example
Dry-Run

Input:

JSON
[
  {
    "purchase_id": 12345,
    "customer_id": "C007",
    "product_name": "Laptop",
    "price": 999.99,
    "timestamp": "2023-10-05T14:00:00Z"
  },
  {
    "purchase_id": 67890,
    "customer_id": "C007",
    "product_name": "Mouse",
    "price": 19.99,
    "timestamp": "2023-10-05T14:30:00Z"
  }
]

Output (in a lakehouse with Delta Lake):

SQL
SELECT customer_id, product_name, SUM(price) as total_spent
FROM purchases
GROUP BY customer_id, product_name
ORDER BY total_spent DESC;

This example demonstrates the ease of performing complex analytics directly on structured data stored in a lakehouse.

**Code:**

```python
# Example Python code to read and query Delta Lake using PySpark

from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("LakehouseExample").getOrCreate()

# Reading the purchases table from Delta Lake
purchases = spark.read.format("delta").load("/path/to/purchases")

# Querying the data
result = purchases.groupBy("customer_id", "product_name").agg({"price": "sum"}).orderBy("sum(price)", ascending=False)

result.show()
```

**Example input / output:**

```sql
-- SQL Query in a lakehouse using Delta Lake
SELECT customer_id, product_name, SUM(price) as total_spent
FROM purchases
GROUP BY customer_id, product_name
ORDER BY total_spent DESC;
customer_id	product_name	total_spent
C007	Laptop	999.99
C007	Mouse	19.99
```

**Data:**

```json
[
  {
    "purchase_id": 12345,
    "customer_id": "C007",
    "product_name": "Laptop",
    "price": 999.99,
    "timestamp": "2023-10-05T14:00:00Z"
  },
  {
    "purchase_id": 67890,
    "customer_id": "C007",
    "product_name": "Mouse",
    "price": 19.99,
    "timestamp": "2023-10-05T14:30:00Z"
  }
]
```

