---
type: "practice"
difficulty: "Intermediate"
---

# How do file formats like CSV versus Parquet change how you think about storing denormalized wide data?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Denormalization in Lakes](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with large datasets, especially in environments where performance and storage efficiency are critical, the choice between CSV (Comma-Separated Values) and Parquet formats can significantly influence how you handle denormalized wide data. This answer will explore how these file formats impact normalization and denormalization strategies.

Firstly, let’s briefly summarize: CSV is a simple text-based format that stores tabular data in plain text, while Parquet is a columnar storage format designed for big data processing frameworks like Apache Hadoop and Spark. CSV files are easily human-readable but less efficient for large datasets due to their textual nature. Parquet, on the other hand, offers better performance and compactness by storing columns of data together.

Now, let’s dive into the specifics:

Normalization in CSV:

#### Pros
Simple to read/write, easy to generate.

#### Cons
Poor for large datasets due to its row-oriented structure, leading to slow query times when querying across multiple rows (denormalized wide data).

Denormalization and Parquet:

#### Pros
Columnar storage allows efficient retrieval of specific columns without loading the entire row, ideal for wide tables.

#### Cons
More complex to write due to its schema requirements.

The key difference lies in how these formats handle denormalized data. Let’s illustrate with an example:

Scenario: Storing Sales Data

Imagine you have a dataset representing daily sales across multiple stores. Each record might include customer details, product information, and sales amounts for different products on that day. This is typically denormalized to aggregate various metrics like total sales per store, top-selling products, etc.

CSV Example:

PLAINTEXT
StoreID, Date, CustomerID, ProductID, SalesAmount, QuantitySold
123456, 2023-09-01, A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6, P7Q8R9S0T1U2V3W4X5Y6Z7, 150.00, 2
123456, 2023-09-01, A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6, P8Q9R0S1T2U3V4W5X6Y7Z8, 200.00, 1

Parquet Example:

PLAINTEXT
StoreID, Date, CustomerID, SalesAmount_1, QuantitySold_1, SalesAmount_2, QuantitySold_2, ...
123456, 2023-09-01, A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6, 150.00, 2, 200.00, 1

Here, each SalesAmount and QuantitySold pair is stored as a separate column.

Step-by-Step Solution:

Normalization:

In CSV, you would read the entire row to get any piece of data.
In Parquet, you can efficiently query specific columns without reading unrelated rows.

Denormalization Considerations:

For CSV, denormalizing might lead to bloated files and slower queries.
For Parquet, denormalization helps in reducing the number of scans required, improving read performance significantly.

Choosing Between Formats:

If you are working with small datasets or non-real-time applications where data is not queried often, CSV might be sufficient.
For large-scale data processing and frequent querying scenarios, Parquet’s denormalized structure offers better performance and scalability.

#### Closing Summary

In summary, when dealing with wide, denormalized data, the choice between CSV and Parquet significantly impacts how you design your data storage strategy. While CSV is simple and human-readable, Parquet's columnar storage optimizes query performance, making it more suitable for large-scale processing environments.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring schema requirements in Parquet, leading to inefficiencies.

#### Follow-Up
Discussing how data quality and consistency are maintained differently between the two formats.

**Comparison:**

| StoreID | Date       | CustomerID         | SalesAmount_1 | QuantitySold_1 |
|---------|------------|--------------------|---------------|----------------|
| 123456  | 2023-09-01 | A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6 | 150.00         | 2              |

**Code:**

```python
# Example of reading CSV with pandas (simplified)
import pandas as pd

csv_data = pd.read_csv('sales.csv')
print(csv_data)

# Example of reading Parquet with pandas
parquet_data = pd.read_parquet('sales.parquet')
print(parquet_data)
```

**Example input / output:**

```text
Input (CSV):
StoreID, Date, CustomerID, ProductID, SalesAmount, QuantitySold
123456, 2023-09-01, A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6, P7Q8R9S0T1U2V3W4X5Y6Z7, 150.00, 2

Output (Parquet):
StoreID: 123456
Date: 2023-09-01
CustomerID: A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6
SalesAmount_1: 150.00
QuantitySold_1: 2
```

**Data:**

```

StoreID, Date, CustomerID, ProductID, SalesAmount, QuantitySold 123456, 2023-09-01, A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6, P7Q8R9S0T1U2V3W4X5Y6Z7, 150.00, 2

CODE
Copy

```plaintext
StoreID, Date, CustomerID, SalesAmount_1, QuantitySold_1, SalesAmount_2, QuantitySold_2
123456, 2023-09-01, A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6, 150.00, 2, 200.00, 1
```

**Flow:**

1. Yes
2. No
3. Read CSV
4. Is data large?
5. Use Parquet for betterperformance
6. Use CSV for simplicity
7. Optimize queries with specificcolumn selection
8. Ensure consistency andschema adherence

